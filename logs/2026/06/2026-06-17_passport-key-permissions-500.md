---
date: 2026-06-17
type: BUG
tech: [Laravel, Passport, Deploy]
repos: [api.sarasavi.lk]
tags: [laravel, passport, deploy, permissions, auth, 500-error]
status: final
---

# Laravel Passport Key Permissions Caused Login Failure

## Problem

Login/API authentication failed after deployment with a server error.

## Context / Reproduction

The deploy process used `git reset --hard origin/main`. Laravel Passport keys live under `storage/` and are gitignored, so the reset removed the keys. A later request triggered key generation, but the generated private key had insecure permissions.

## What I Tried

Checked Passport key existence, deploy script behavior, and file permissions around `storage/oauth-private.key`.

## Root Cause

The deploy script only hardened key permissions inside this kind of condition:

```bash
if [ -f storage/oauth-private.key ]; then
  chmod 600 storage/oauth-private.key
fi
```

After `git reset --hard`, the key did not exist when the permission block ran. Passport then generated the key later with default permissions that were readable by group/world, and `league/oauth2-server` rejected it as insecure.

## Fix

Immediate fix:

```bash
cd /www/wwwroot/api.sarasavi.org

php artisan passport:keys --force
chmod 600 storage/oauth-private.key
chmod 644 storage/oauth-public.key
chown www:www storage/oauth-private.key storage/oauth-public.key
```

Permanent deploy fix:

```bash
if [ ! -f storage/oauth-private.key ] || [ ! -f storage/oauth-public.key ]; then
  php artisan passport:keys --force
fi

chmod 600 storage/oauth-private.key
chmod 644 storage/oauth-public.key
chown www:www storage/oauth-private.key storage/oauth-public.key
```

Move key generation before migrations and before any endpoint health check that uses Passport.

## Verification

```bash
stat -c %a storage/oauth-private.key
curl -I https://api.sarasavi.org
```

Expected private key permission:

```text
600
```

Login returned `200 OK` after the fix.

## Lesson

Never assume gitignored runtime keys survive deploys; regenerate missing Passport keys before hardening permissions.

## Impact

Restored login/API authentication and improved the deploy script so missing Passport keys are repaired deterministically.

## Resume / Interview Bullet

Diagnosed and fixed a Laravel Passport authentication failure by tracing deploy-time key deletion and enforcing secure key generation and permissions in the release script.

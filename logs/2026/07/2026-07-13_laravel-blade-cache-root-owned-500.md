---
date: 2026-07-13
type: BUG
tech: [Laravel, aaPanel, PHP-FPM]
repos: [www.sarasavi.lk]
tags: [laravel, deploy, blade-cache, permissions, 500-error]
status: final
---

# Root-Owned Blade Cache Caused 500 After Deploy

## Problem

The public site returned HTTP 500 after deploying `v20260713.3`.

## Context / Reproduction

Laravel failed while rendering Blade views after deployment.

Error:

```text
touch(): Utime failed: Operation not permitted
```

## What I Tried

Checked the Laravel render path and identified that the failure happened inside the Blade compiler when it attempted to update compiled view files.

## Root Cause

Deployment cache commands had been run as `root`. That created root-owned compiled Blade files under `storage/framework/views`. PHP-FPM runs as a different user, so it could not update those compiled files during page rendering.

## Fix

Restore ownership of Laravel writable paths to the PHP-FPM user, then rebuild caches as that same user.

```bash
cd /www/wwwroot/www.sarasavi.lk

chown -R www:www storage bootstrap/cache
chmod -R 775 storage bootstrap/cache

sudo -u www php artisan optimize:clear
sudo -u www php artisan config:cache
sudo -u www php artisan route:cache
sudo -u www php artisan view:cache
```

## Verification

Confirmed the public site no longer returned HTTP 500 and Laravel could render Blade pages without the `touch(): Utime failed` error.

## Lesson

Run Laravel cache commands as the PHP-FPM/runtime user, never as `root`.

## Impact

Restored the public site after deploy and documented the permission pattern needed to prevent repeat Laravel cache failures.

## Resume / Interview Bullet

Diagnosed and resolved a production Laravel 500 error by tracing Blade compiler permission failures to root-owned deploy artifacts and restoring runtime-safe cache ownership.

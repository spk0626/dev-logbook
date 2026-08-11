---
date: 2026-08-11
type: DEPLOY
tech: [Laravel, aaPanel, Git, Nginx, PHP-FPM]
repos: [kiosk.sarasavi.lk]
tags: [aapanel, laravel, deploy, kiosk, runbook]
status: reference
---

# aaPanel Laravel Deploy Procedure

## Problem

Need a repeatable aaPanel deployment procedure for a Laravel app such as the Kiosk project.

## Context / Reproduction

This is a reusable runbook, not a single outage. It records the expected production setup and future deploy flow.

## What I Tried

Captured the manual aaPanel setup, Git pull, environment configuration, dependency install, build, migration, permissions, cache, admin creation, and verification steps.

## Root Cause

Laravel deployments on aaPanel fail easily when the site root, PHP version, runtime user, cache permissions, storage link, or deploy flow are inconsistent.

## Fix

Create the site in aaPanel:

```text
Website -> Add site
Domain: kiosk.sarasavi.lk
Root: /www/wwwroot/kiosk.sarasavi.lk/public
PHP: 8.3
SSL: enable after DNS points correctly
```

Clone or connect the Git repo:

```bash
cd /www/wwwroot
git clone git@github.com:Sarasavi-Bookshop/kiosk.sarasavi.lk.git kiosk.sarasavi.lk
cd /www/wwwroot/kiosk.sarasavi.lk
```

If using an aaPanel deploy key:

```bash
GIT_SSH_COMMAND='ssh -i ~/.ssh/aapanel/kiosk_08009 -o IdentitiesOnly=yes' git pull origin main
```

Create `.env`:

```bash
cp .env.example .env
nano .env
```

Minimum production values:

```ini
APP_NAME="Sarasavi Kiosk"
APP_ENV=production
APP_DEBUG=false
APP_URL=https://kiosk.sarasavi.lk

DB_CONNECTION=mysql
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=your_db
DB_USERNAME=your_user
DB_PASSWORD=your_password

SESSION_DRIVER=database
SESSION_TABLE=kiosk_sessions
SESSION_DOMAIN=
SESSION_SECURE_COOKIE=true
SESSION_SAME_SITE=lax

FILESYSTEM_DISK=public
CACHE_STORE=database
QUEUE_CONNECTION=database
KIOSK_DISPLAY_URL=https://kiosk.sarasavi.lk
```

Generate app key:

```bash
php artisan key:generate --force
```

Install PHP dependencies:

```bash
COMPOSER_ALLOW_SUPERUSER=1 composer install --no-dev --no-interaction --prefer-dist --optimize-autoloader
```

Fallback with explicit PHP:

```bash
COMPOSER_ALLOW_SUPERUSER=1 /www/server/php/83/bin/php /usr/bin/composer install --no-dev --no-interaction --prefer-dist --optimize-autoloader
```

Install frontend dependencies and build:

```bash
npm ci
npm run build
```

Run migrations and link storage:

```bash
php artisan migrate --force
php artisan storage:link
```

Set permissions:

```bash
chown -R www:www storage bootstrap/cache
chmod -R 775 storage bootstrap/cache
```

Cache production config:

```bash
php artisan optimize:clear
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

Create the first admin:

```bash
php artisan kiosk:create-admin "admin" admin@example.test
```

Fallback via Tinker with placeholder credentials:

```bash
php artisan tinker --execute='App\Models\User::updateOrCreate(["email" => "admin@example.test"], ["name" => "admin", "email_verified_at" => now(), "password" => Illuminate\Support\Facades\Hash::make("CHANGE_ME")]);'
```

Future deploy:

```bash
cd /www/wwwroot/kiosk.sarasavi.lk

git status --short
git diff

GIT_SSH_COMMAND='ssh -i ~/.ssh/aapanel/kiosk_08009 -o IdentitiesOnly=yes' git pull origin main

COMPOSER_ALLOW_SUPERUSER=1 composer install --no-dev --no-interaction --prefer-dist --optimize-autoloader

npm ci
npm run build

php artisan migrate --force
php artisan optimize:clear
php artisan config:cache
php artisan route:cache
php artisan view:cache

chown -R www:www storage bootstrap/cache
chmod -R 775 storage bootstrap/cache
```

## Verification

```bash
php artisan about
php artisan migrate:status
ls -la public/storage
curl -I https://kiosk.sarasavi.lk/login
```

## Lesson

Never edit deployed code directly on the server; change locally, commit, push, then pull on the server.

## Impact

Created a reusable aaPanel Laravel deployment checklist that reduces missed setup steps and permission-related production failures.

## Resume / Interview Bullet

Documented a repeatable Laravel aaPanel deployment procedure covering Git deployment, environment setup, migrations, permissions, caching, and production verification.

---
date: 2026-06-17
type: BUG
tech: [Laravel, aaPanel, Nginx, Apache, PHP-FPM]
repos: [api.sarasavi.lk]
tags: [api, aapanel, nginx, apache, anti-xss, open-basedir, 421-error]
status: final
---

# Apache/Nginx Split-Brain And Anti-XSS Broke API

## Problem

The API returned a misdirected request error.

```text
GET https://api.sarasavi.org/api/geo-records -> 421 Misdirected Request
Server: Apache/2.4.x
```

There was also an API failure described as "No input file found".

## Context / Reproduction

aaPanel had configured the site through Apache on backend ports `8288` and `8290`, while Nginx reverse proxied HTTPS traffic to Apache.

## What I Tried

Checked the web server path, aaPanel website settings, PHP restrictions, and deploy script target directory.

## Root Cause

Three issues overlapped:

- Apache vhost `ServerName` did not match SNI for `api.sarasavi.org`, causing HTTP 421.
- aaPanel Anti-XSS re-enabled `open_basedir`, restricting PHP to `/public:/tmp/` and breaking Laravel access to `vendor`.
- The deploy script defaulted to the `.lk` API directory instead of the `.org` API directory, so a deploy command modified the wrong path.

## Fix

Use Nginx directly, disable Anti-XSS for Laravel, and deploy to the correct app directory.

```bash
# aaPanel UI
# Website -> api.sarasavi.org -> Web Server -> Nginx
# Website -> api.sarasavi.org -> Directory -> Anti-XSS attack -> OFF

APP_DIR=/www/wwwroot/api.sarasavi.org ./deploy-api.sh

systemctl reload php-fpm-83
systemctl reload nginx
```

## Verification

```bash
ss -ltnp | grep :443
curl -I https://api.sarasavi.org
```

Expected:

```text
nginx serves 443 directly
HTTP 200
```

## Lesson

Never use Apache plus aaPanel for this Laravel API setup; keep the site on Nginx and keep Anti-XSS off for Composer/Laravel apps.

## Impact

Restored API availability and documented the aaPanel configuration combination that caused SNI, Laravel autoloading, and deploy-target failures.

## Resume / Interview Bullet

Resolved a production API outage by isolating aaPanel Apache/Nginx SNI mismatch, Laravel `open_basedir` restrictions, and deploy script directory drift.

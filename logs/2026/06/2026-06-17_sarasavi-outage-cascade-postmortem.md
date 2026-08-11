---
date: 2026-06-17
type: INVESTIGATION
tech: [Laravel, Next.js, aaPanel, Nginx, PHP-FPM, PM2]
repos: [api.sarasavi.lk, cms.sarasavi.lk, www.sarasavi.lk]
tags: [postmortem, outage, aapanel, nginx, anti-xss, nextjs, 421-error, 502-error]
status: final
---

# Sarasavi Outage Cascade Postmortem

## Problem

Between June 15 and June 17, multiple production services failed in sequence:

- API returned 500.
- Web returned 502.
- Frontend showed client-side exceptions.
- API and CMS returned 421.

## Context / Reproduction

Observed failures:

```text
api.sarasavi.org -> 500
sarasavi.org -> 502 Bad Gateway
frontend -> Application error: client-side exception
api.sarasavi.org and cms.sarasavi.org -> 421 Misdirected Request
```

## What I Tried

Separated the outage by layer:

- Laravel/PHP-FPM restrictions
- Next.js build ownership and PM2 runtime
- API SSL/SNI behavior
- aaPanel-generated Nginx vhosts
- frontend null-safety after failed API responses

## Root Cause

The outage was a cascade of server configuration and deployment mistakes:

- aaPanel Anti-XSS added `open_basedir` restrictions that blocked Laravel from loading `vendor`, `storage`, and `bootstrap`.
- `.next` was deleted during debugging, then rebuilt as `root`, making it unreadable by the `www` PM2 process.
- `api.sarasavi.org` lacked a correct SSL vhost/SNI setup, returning 421 instead of API JSON.
- Manual Nginx edits mixed HTTP/2 and SSL directives across vhosts and created further 421 behavior.
- Frontend code treated failed/empty API responses as valid data and crashed on undefined values.

## Fix

Laravel Anti-XSS/open_basedir:

```bash
cd /www/wwwroot/api.sarasavi.org

# aaPanel UI:
# Website -> api.sarasavi.org -> Directory
# Running directory: /public
# Anti-XSS attack: OFF

chattr -i public/.user.ini
rm public/.user.ini
systemctl reload php-fpm-83
```

Next.js build ownership:

```bash
cd /www/wwwroot/www.sarasavi.lk

chown -R www:www .
sudo -H -u www npm run build
sudo -u www PM2_HOME=/home/www/.pm2 pm2 restart sarasavi-web-org
sudo -u www PM2_HOME=/home/www/.pm2 pm2 save
```

SSL/SNI:

```text
aaPanel -> Website -> api.sarasavi.org -> SSL
Disable old SSL
Issue Let's Encrypt SSL

aaPanel -> Website -> cms.sarasavi.org -> SSL
Disable old SSL
Issue Let's Encrypt SSL
```

Frontend hardening:

```text
Default list state to [].
Use optional chaining for API response data.
Handle failed API requests without crashing the page.
```

## Verification

```bash
curl -I https://sarasavi.org
curl -I https://api.sarasavi.org
curl -I https://cms.sarasavi.org
curl -sf https://api.sarasavi.org/api/health
```

Expected all services to return HTTP 200 or valid API JSON, not 421/500/502 HTML error pages.

## Lesson

Debug production outages by layer first: web server, PHP/runtime, app build, API response shape, then frontend rendering.

## Future Watch-Outs

- aaPanel Anti-XSS can silently inject `open_basedir` restrictions through `public/.user.ini`, breaking Laravel access to Composer and framework paths.
- `502 Bad Gateway` with PM2 is often a process, build, or runtime-user issue such as missing `.next` files or wrong ownership.
- `421 Misdirected Request` is usually an Nginx/SNI/vhost issue, not application code.
- Browser CORS errors can be symptoms of an API returning an HTML 421/50x page instead of JSON; confirm `200` plus JSON before changing frontend code.
- Manual edits under `/www/server/panel/vhost/nginx/*.conf` can be overwritten by aaPanel and cause split-brain configuration.

## Impact

Restored API, CMS, and Web service availability and created a reusable checklist for future aaPanel Laravel/Next.js outages.

## Resume / Interview Bullet

Led root-cause analysis of a multi-service production outage by separating aaPanel, Nginx, Laravel, PM2, and frontend failure modes, restoring service and documenting prevention steps.

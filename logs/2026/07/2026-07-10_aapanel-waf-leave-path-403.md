---
date: 2026-07-10
type: BUG
tech: [aaPanel, Nginx, SSL]
repos: []
tags: [aapanel, nginx, waf, ssl, 403-error]
status: final
---

# aaPanel WAF Blocked Leave Path After SSL

## Problem

`https://www.sarasavi.org/leave/` returned `403 Forbidden` after SSL was enabled.

## Context / Reproduction

Symptoms:

- `/leave/` returned 403 only over the SSL site.
- Root domain loaded correctly.
- Apache access and error logs stayed empty.
- File permissions were verified as `755`.
- Owner was verified as `www:www`.
- PHP mode was set to Pure Static.
- No matching rules existed in aaPanel Limit Access or URL Rewrite tabs.

## What I Tried

Checked Apache config, file permissions, ownership, PHP mode, aaPanel access rules, and rewrite rules before identifying that the request never reached Apache.

## Root Cause

aaPanel enabled the Nginx WAF module `ngx_lua_waf` by default on the new SSL site. Its default ruleset included a `^/leave` pattern meant to block HR/employee leave system paths. Nginx blocked the request before proxying to the Apache backend on ports `8288` or `8290`.

## Fix

Rename the public path so it no longer matches the WAF rule.

```text
aaPanel -> Files
Rename: /leave/
To:     /leave-form/
```

No WAF rules were disabled.

## Verification

```bash
curl -I https://www.sarasavi.org/leave-form/
```

Expected result:

```text
HTTP/2 200
```

SSL remained valid, and the renamed path loaded `index.html`.

## Lesson

When Apache logs are empty for a 403 behind aaPanel, check the Nginx WAF layer before changing app files or permissions.

## Impact

Restored access without weakening WAF security and documented a hidden aaPanel default rule that can block common business terms.

## Resume / Interview Bullet

Diagnosed and resolved an SSL-only 403 by isolating an aaPanel Nginx WAF rule before the Apache backend and applying a low-risk path rename instead of disabling security controls.

---
date: 2026-06-22
type: INVESTIGATION
tech: [aaPanel, Nginx, DNS]
repos: [api.sarasavi.lk, cms.sarasavi.lk, www.sarasavi.lk]
tags: [dns, aapanel, nginx, cname, split-horizon, routing]
status: blocked
---

# DNS And IP Misroute Across Six Domains

## Problem

Six domains across API, CMS, and Web for `.org` and `.lk` were not routing correctly.

- `.org` domains timed out for public users.
- `.lk` domains loaded the live/prod app instead of the test app on this server.

## Context / Reproduction

Target aaPanel public IP:

```text
124.43.4.86
```

Observed DNS:

```text
Internal DNS: api.sarasavi.lk  -> 192.168.200.12
Public DNS:   api.sarasavi.lk  -> 124.43.20.102
Public DNS:   api.sarasavi.org -> 124.43.22.241
```

The target public IP was not present in the public records.

## What I Tried

Validated public DNS, internal DNS behavior, Nginx vhosts, and local vhost routing.

```bash
dig +short api.sarasavi.lk @8.8.8.8
nslookup api.sarasavi.org 1.1.1.1
nginx -T | grep server_name
curl -I http://cms.sarasavi.lk --resolve cms.sarasavi.lk:80:127.0.0.1
```

## Root Cause

Multiple routing issues overlapped:

- Split-horizon DNS made internal testing resolve to `192.168.200.12`, which worked only on LAN.
- Public DNS pointed to other public IPs instead of the intended aaPanel server.
- `.org` and `.lk` subdomains were chained by CNAME to `api.sarasavi.lk`, causing all traffic to arrive with the API host mapping before separate vhosts existed.
- The private IP `192.168.200.12` is not routable for public users, creating false-positive internal tests.

## Fix

Required DNS/admin changes:

```text
Update A records for all six domains to 124.43.4.86.
Remove the CNAME chain to api.sarasavi.lk.
Verify firewall NAT maps 124.43.4.86:80/443 to 192.168.200.12:80/443.
```

No code changes were needed.

## Verification

Nginx vhost config was validated as ready. Final public verification was blocked until DNS and firewall changes were completed by the DNS/admin owner.

## Lesson

Always validate public DNS with `dig @8.8.8.8` or another external resolver because internal DNS can hide production routing issues.

## Impact

Separated app/server configuration from DNS ownership and prevented wasted debugging in Laravel, Next.js, and aaPanel when the real blocker was external routing.

## Resume / Interview Bullet

Investigated a six-domain routing failure by comparing internal and public DNS, identifying split-horizon records and CNAME masking, and isolating the fix to DNS and firewall ownership.

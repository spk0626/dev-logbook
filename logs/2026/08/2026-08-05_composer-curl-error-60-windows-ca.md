---
date: 2026-08-05
type: LEARNING
tech: [Laravel, Composer, PHP, Windows]
repos: []
tags: [composer, curl, ssl, windows, ca-certificate, laravel]
status: final
---

# Composer Curl Error 60 On Windows CA Trust

## Problem

Creating a new Laravel project failed during Composer package download.

```bash
composer create-project laravel/laravel book-store
```

Error:

```text
curl error 60 while downloading https://repo.packagist.org/packages.json:
SSL peer certificate or SSH remote key was not OK
```

## Context / Reproduction

This happened on Windows while Composer tried to download packages from Packagist.

## What I Tried

Separated the error by type:

- DNS issues usually show host resolution errors.
- Connectivity issues usually show timeout or connection refused.
- Auth issues usually show 401 or 403.
- Curl error 60 points to certificate trust.

## Root Cause

PHP/Composer on Windows did not have a valid CA bundle configured, or HTTPS inspection from antivirus/proxy/network tooling interfered with certificate validation.

## Fix

Check Composer and PHP SSL configuration:

```bash
composer diagnose
php --ini
php -i | findstr /i "curl cafile openssl"
```

Verify:

```text
OpenSSL enabled
curl enabled
curl.cainfo configured
openssl.cafile configured
Composer does not report certificate trust problems
```

Configure a CA bundle in `php.ini`:

```ini
curl.cainfo = C:\path\to\cacert.pem
openssl.cafile = C:\path\to\cacert.pem
```

Then restart the terminal and retry:

```bash
composer clear-cache
composer create-project laravel/laravel book-store
```

Also check simple external causes:

```text
Confirm Windows date/time is correct.
Try another network or hotspot.
Temporarily disable HTTPS scanning/SSL inspection in antivirus or proxy tools.
```

## Verification

Composer should download `https://repo.packagist.org/packages.json` without curl error 60 and complete the Laravel project creation.

## Lesson

Treat Composer curl error 60 as a TLS trust problem first, not a Laravel or database problem.

## Impact

Created a repeatable Windows Composer SSL checklist and clarified that Laravel setup is separate from installing/configuring MySQL.

## Resume / Interview Bullet

Diagnosed Composer package installation failures on Windows by identifying TLS certificate trust errors and documenting PHP CA bundle configuration for reliable Laravel setup.

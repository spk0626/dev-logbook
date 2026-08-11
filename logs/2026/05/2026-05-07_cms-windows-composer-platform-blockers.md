---
date: 2026-05-07
type: INVESTIGATION
tech: [Laravel, Composer, Windows, WSL, Redis]
repos: [cms.sarasavi.lk, api.sarasavi.lk]
tags: [cms, laravel, composer, windows, pcntl, posix, horizon, antivirus]
status: final
---

# CMS Could Not Boot Natively On Windows

## Problem

The CMS project could not be started in the Windows session.

## Context / Reproduction

There was no `.env` file in the workspace, but that was not the immediate blocker. Dependency installation failed before the app could boot.

## What I Tried

Attempted Composer installation normally, then retried with platform requirement ignores for Linux-only extensions.

## Root Cause

Two blockers existed:

- `laravel/horizon` requires Linux-only extensions such as `pcntl` and `posix`, which are not available in native Windows PHP.
- After bypassing platform requirements, package downloads from GitHub still timed out or failed SSL checks because antivirus/proxy/firewall behavior interfered with Composer downloads.

## Fix

Temporary Windows install commands:

```bash
# API
composer install --ignore-platform-req=ext-pcntl --ignore-platform-req=ext-posix

# CMS
composer install --ignore-platform-req=ext-pcntl --ignore-platform-req=ext-posix --ignore-platform-req=ext-sodium
```

If Composer downloads fail:

```text
Temporarily disable HTTPS scanning or antivirus interference.
Use a faster Wi-Fi or hotspot.
Retry composer install.
```

Permanent development approach:

```text
Use WSL2 with Ubuntu.
Install PHP, Composer, MySQL, and Redis inside WSL.
Clone the repo inside the WSL filesystem.
Create the project .env from server values or a local template.
Run Composer, migrations, queues, and Laravel commands inside WSL.
Use VS Code Remote - WSL for editing and execution.
```

## Verification

Composer dependencies should install successfully and Laravel commands should run in a Linux-like environment where `pcntl`, `posix`, Redis, and Horizon expectations match production more closely.

## Lesson

Run Laravel apps that depend on Horizon and Linux extensions in WSL/Linux instead of native Windows.

## Impact

Identified the correct local development environment and avoided wasting time trying to force a production-like Laravel CMS stack into native Windows PHP.

## Resume / Interview Bullet

Investigated Laravel CMS local startup failures by separating missing environment configuration from Composer platform blockers and recommending a WSL-based development workflow aligned with production.

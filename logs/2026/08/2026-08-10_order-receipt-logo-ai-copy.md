---
date: 2026-08-10
type: BUG
tech: [Laravel, Blade, Email]
repos: [api.sarasavi.lk]
tags: [email, order-receipt, logo, copy]
status: final
---

# Order Receipt Email Logo Did Not Render And Copy Sounded AI-Written

## Problem

The order receipt email logo did not render reliably. The email heading and copy also sounded too generic and AI-written.

## Context / Reproduction

The order receipt email is rendered from:

```text
resources/views/email/order-place.blade.php
```

## What I Tried

Checked how the email template built the logo URL and reviewed the customer-facing text in the email header.

## Root Cause

The template depended on an external CMS asset URL for the logo. Email clients can be strict with remote assets, so the logo path was fragile.

The copy used generic wording instead of plain order-confirmation language.

## Fix

Embedded the local logo when the mail message object is available, with an asset-base URL fallback.

```php
$assetBaseUrl = rtrim(config('app.cms_asset_base_url', config('app.url')), '/');
$logoPath = public_path('images/logo.png');
$logoSrc = isset($message) && file_exists($logoPath)
    ? $message->embed($logoPath)
    : $assetBaseUrl . '/images/logo.png';
```

Updated the alt text and receipt copy to use plain customer-facing wording.

## Verification

Reviewed the Blade template so the logo has a local embed path plus fallback, and the email copy no longer uses generic AI-style wording.

## Lesson

Transactional emails need reliable asset handling and simple human copy.

## Impact

Improved order receipt presentation without changing order behavior.

## Resume / Interview Bullet

Improved transactional order email reliability by adding embedded logo fallback handling and replacing generic receipt copy with clearer customer-facing text.

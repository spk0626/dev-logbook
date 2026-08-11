---
date: 2026-08-06
type: FEATURE
tech: [Next.js, Laravel CMS, Checkout, Playwright]
repos: [www.sarasavi.lk, cms.sarasavi.lk]
tags: [checkout, payment-discounts, cms-promotions, boc, koko, payhere]
status: final
---

# Payment Method Discounts Needed CMS Rules Plus Defaults

## Problem

Checkout payment method discounts were partly hidden in Web code. The rules needed to move toward CMS-managed promotions, but the old default BOC, PayHere, and Koko discounts still had to work when no CMS rule matched.

## Context / Reproduction

Checkout and product pages displayed payment method discounts for:

```text
BOC
PayHere
Koko
```

The first pass moved the rules to CMS `cart_payment_method` promotions. That made the system configurable, but it also meant expected discounts could disappear if CMS rules were missing or incomplete.

## What I Tried

Compared the old Web-only discount behavior with the new CMS promotion evaluator. Checked delivery checkout, click-and-collect checkout, and product payment preview behavior.

## Root Cause

The Web checkout had hardcoded payment discount behavior. Moving to CMS-only rules removed the default discount path unless production CMS data was prepared before deploy.

## Fix

Added a shared Web payment discount evaluator that supports CMS `cart_payment_method` promotions and payment discount buckets.

Then restored default payment method rules inside the evaluator:

```text
BOC: 20%
PayHere: 15%
Koko: 10%
```

CMS rules can still win when they provide a higher matching discount. Discounts do not stack.

## Verification

Playwright coverage checks:

- CMS payment discount bucket is used instead of hidden Web rates.
- Default payment method discounts apply when CMS has no matching rule.
- Click-and-collect keeps online payment methods but hides payment method discounts.

## Lesson

When moving business rules into CMS, keep a safe default path until production data is guaranteed.

## Impact

Made payment discounts configurable while keeping expected checkout totals for customers.

## Resume / Interview Bullet

Refactored checkout payment discounts into a shared evaluator backed by CMS rules and safe defaults, with Playwright coverage for delivery and collect flows.

---
date: 2026-05-18
type: BUG
tech: [Next.js, Redux, Checkout]
repos: [www.sarasavi.lk]
tags: [checkout, shipping, payments, koko, payhere, validation]
status: final
---

# Checkout Could Submit Before Shipping Was Ready

## Problem

Shipping orders could be created with `shippingAmount = 0` when the shipping price had not finished calculating.

The checkout UI and create-order payload also used different subtotal sources, so Koko/PayHere rows could be stored with totals that did not match the customer-facing display.

## Context / Reproduction

The checkout button logic was too tied to the raw shipping amount. A stale or missing shipping value could be mistaken for a valid one.

Browser validation was noisy because the local bundle was sometimes stale after edits, so a few checks were accidentally run against older code before the refresh/reload issue was noticed.

## What I Tried

Compared the checkout display total, payment page total, create-order payload, and button enable/disable condition. Then checked whether the same shipping readiness rule was used across checkout and payment flows.

## Root Cause

The shipping readiness rule was duplicated and too implicit:

- `0` could mean "not calculated yet", but could also become a valid value if free shipping is introduced.
- Checkout and payment pages could drift because each had its own condition.
- Payload totals and display totals were not guaranteed to use the same subtotal source.

## Fix

Use the same subtotal source in the payload as the value shown in the UI, and block submit until shipping is explicitly ready.

Extract the shipping rule into a shared helper:

```text
lib/shipping.js
pages/checkout.js
pages/payment.js
```

Replace ad hoc checks with one shared guard expression so both pages use the same condition.

Expected behavior:

```text
If shipping is still calculating, disable order submission.
If shipping is valid, use the same computed total for display and create-order payload.
Do not treat raw 0 alone as proof that shipping is ready or not ready.
```

## Verification

Verify these flows:

```text
Select a city with a shipping fee.
Confirm the checkout display total matches the create-order payload total.
Confirm Koko/PayHere stored total matches the displayed total.
Refresh the browser after code edits and repeat the test with a fresh bundle.
```

## Lesson

Treat `0` as a value, not a readiness signal; base submit eligibility on explicit shipping readiness plus the current selected city and computed fee.

## Impact

Prevented incorrect zero-shipping orders and reduced payment/order total mismatches between the UI, create-order payload, and payment records.

## Resume / Interview Bullet

Fixed checkout total mismatches by centralizing shipping readiness logic and aligning payment payload totals with the UI, preventing zero-shipping orders caused by stale calculation state.

---
date: 2026-08-07
type: BUG
tech: [Laravel, Next.js, CMS]
repos: [api.sarasavi.lk, cms.sarasavi.lk, www.sarasavi.lk]
tags: [promotions, discount-badge, product-card, cms]
status: final
---

# Promotion Badge Text Feature Broke Discount Badges

## Problem

After deploying the new promotion badge text feature, discount badges started disappearing or did not show the custom text entered from the CMS.

Earlier, badges showed calculated numbers. The new feature let admins enter label text through CMS, but that text was only for display and should not have changed discount amount calculations.

## Context / Reproduction

The feature touched all three layers:

```text
cms.sarasavi.lk promotion form and repository
api.sarasavi.lk promotion discount calculation
www.sarasavi.lk product cards
```

## What I Tried

Checked whether CMS saved the badge fields, whether API returned them with promotion data, and whether Web rendered the badge when the mode was automatic, custom, or hidden.

## Root Cause

Badge display data was added in CMS, but API and Web did not carry and render `badgeDisplay` and `discountBadgeText` in a consistent way.

The badge metadata also needed to come from the same promotion that won the discount calculation. Otherwise the badge could describe the wrong promotion or disappear.

## Fix

Added CMS fields for badge display mode and custom badge text:

```text
badgeDisplay
discountBadgeText
```

Updated API promotion calculation so badge metadata comes from the winning promotion.

Updated Web product cards to:

- normalize missing or invalid badge display mode to `automatic`
- show custom text only when mode is `custom`
- hide the badge only when mode is `hidden`
- keep discount math separate from badge text

## Verification

Checked the CMS to API to Web flow and confirmed badge display behavior works for automatic, custom, and hidden modes.

## Lesson

When adding display-only CMS fields, keep them separate from calculation logic and pass them with the exact rule that produced the displayed result.

## Impact

Restored discount badges and gave admins control over badge text without changing discount amount calculations.

## Resume / Interview Bullet

Fixed promotion badge display across CMS, API, and Web by carrying badge metadata with the winning discount rule and keeping display text separate from discount math.

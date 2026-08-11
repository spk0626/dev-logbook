---
date: 2026-08-06
type: BUG
tech: [Laravel, Cache, Promotions]
repos: [api.sarasavi.lk]
tags: [best-sellers, promotions, cache, product-api]
status: final
---

# Best Sellers Disappeared After Release Because Promotion Data Was Cached

## Problem

Best Sellers suddenly disappeared after a release, or looked like they disappeared from the home page. The suspected trigger was stale promotion data inside the cached Best Sellers response.

## Context / Reproduction

Home Best Sellers uses:

```text
GET /api/get-amazon-best-seller-products
```

The endpoint is backed by local `Amazon_top_100` rows and returns decorated product data for the Web home page.

## What I Tried

Checked the Best Sellers endpoint and how it cached products, gallery data, and promotion data.

## Root Cause

The endpoint cached the decorated product result, including promotion data. When CMS promotions changed, the cached Best Sellers payload could keep old promotion values until the product cache expired.

That made promotion changes look broken on the home page and may have contributed to the Best Sellers section disappearing after release.

## Fix

Kept the ranked Best Sellers product list in cache, but moved promotion calculation outside the cache read.

The endpoint now:

```text
1. Reads the cached ranked product list.
2. Calculates current promotion data after the cache read.
3. Returns products with fresh promotion values.
```

## Verification

Developer notes were updated to document that Best Sellers caches the ranked product list, while promotions are attached after the cache read.

## Lesson

Do not cache fast-changing promotion or pricing data together with slower-changing product lists.

## Impact

Allowed CMS promotion changes to appear on Best Sellers without waiting for the product list cache to expire.

## Resume / Interview Bullet

Fixed stale Best Seller promotion displays by separating cached product ranking from live promotion calculation in the API response.

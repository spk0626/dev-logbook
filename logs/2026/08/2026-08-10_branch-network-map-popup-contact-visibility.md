---
date: 2026-08-10
type: FEATURE
tech: [Next.js, React Bootstrap, SCSS]
repos: [www.sarasavi.lk]
tags: [branch-network, maps, contact-hub, ui, ux]
status: final
---

# Branch Network Page Hid Useful Contact Details Too Far Down

## Problem

Branch contacts were not visible enough at first glance. Customers had to scroll and the map/tab layout competed with the contact details.

## Context / Reproduction

The branch network page had branch buttons and map tab content in the same flow. The page also needed clearer wording that web-order contacts are only for website orders.

## What I Tried

Reviewed the branch network layout and separated the job of browsing contact details from the job of opening a map.

## Root Cause

The layout mixed branch contact browsing and map display. That pushed useful phone, email, and WhatsApp details down the page and made the first view less helpful.

## Fix

Rearranged the branch network page so branch contacts are more visible first.

Moved branch maps into a popup from the branch card:

```text
pages/branch-network.js
styles/globals.scss
```

Kept the selected branch highlighted after the popup closes.

Added a warning under Web Orders explaining that those contacts are only for website orders and branch issues should use branch-specific contact numbers.

## Verification

Developer notes were updated to document the popup map behavior and web-order contact warning.

## Lesson

For contact pages, show phone, email, and WhatsApp details first. Maps are useful, but they should not hide the contact information.

## Impact

Made branch contact browsing clearer and reduced confusion between website order support and branch-level contact details.

## Resume / Interview Bullet

Improved branch contact UX by moving maps into focused popups and clarifying web-order versus branch-specific support channels.

---
date: 2026-08-10
type: BUG
tech: [Laravel, Blade, CMS, AdminLTE]
repos: [cms.sarasavi.lk]
tags: [support-tickets, cms, ui, admin]
status: final
---

# CMS Support Ticket Summary Was Cropped For Long Messages

## Problem

The CMS support ticket detail page cropped the summary block at the bottom when the customer wrote a long issue description.

## Context / Reproduction

The first customer message appears in the primary issue summary before the conversation section. Long summaries were cut off there.

## What I Tried

Checked the ticket detail Blade styles around the primary summary message.

## Root Cause

The summary block had a fixed `max-height` and `overflow: hidden`, so the browser hid the bottom of long messages.

## Fix

Removed the height cap and hidden overflow from the primary summary message block.

```text
resources/views/admin/support_tickets/show.blade.php
```

## Verification

Reviewed the ticket detail page behavior so the full first customer message remains visible in the summary block.

## Lesson

Support screens should show the full customer issue. Compact UI is useful only when it does not hide the main problem.

## Impact

Support admins can read the full reported issue before replying or assigning the ticket.

## Resume / Interview Bullet

Improved CMS support workflow usability by removing clipped ticket summaries so agents can read full customer issue reports before responding.

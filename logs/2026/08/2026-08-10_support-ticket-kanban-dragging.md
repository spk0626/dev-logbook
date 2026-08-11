---
date: 2026-08-10
type: BUG
tech: [Laravel, Blade, jQuery UI, CMS]
repos: [cms.sarasavi.lk]
tags: [support-tickets, cms, kanban, ui]
status: final
---

# Support Ticket Kanban Board Dragging Did Not Work Properly

## Problem

Dragging support tickets on the CMS Kanban board did not work well. The card looked draggable, but most of the card did nothing.

## Context / Reproduction

Support admins use the board to move tickets between status columns. Only a small handle worked for dragging.

## What I Tried

Checked the board sortable setup and the card click behavior.

## Root Cause

The sortable behavior was tied to `.st-card-drag-handle`, so the card itself was not the drag target.

## Fix

Allowed dragging from the whole ticket card and added a small movement threshold so normal clicks still open the ticket detail page.

```javascript
distance: 6
```

## Verification

CMS docs were updated to state that board cards can be dragged by the whole card while normal clicks still open details.

## Lesson

Kanban cards should be easy to drag from the full card area, while normal clicks should still behave like links.

## Impact

Made ticket status updates faster and less frustrating for support admins using the board view.

## Resume / Interview Bullet

Improved CMS support board usability by allowing whole-card drag-and-drop while preserving normal ticket detail clicks.

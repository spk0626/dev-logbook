---
date: 2026-08-10
type: BUG
tech: [Laravel, CMS, DataTables]
repos: [api.sarasavi.lk, cms.sarasavi.lk]
tags: [support-tickets, routing, email-links, cms, idor, security]
status: final
---

# Support Ticket URLs Used Database IDs Instead Of Ticket Numbers

## Problem

Support ticket links in emails and the CMS used database IDs instead of ticket numbers.

This was an IDOR risk. IDOR means Insecure Direct Object Reference. It happens when a URL exposes an internal identifier like:

```text
/support-tickets/123
```

If authorization checks miss a case, someone can try changing `123` to another number and access a record they should not see. Public ticket numbers do not replace authorization, but they reduce ID guessing and keep internal database IDs out of URLs.

## Context / Reproduction

Admin notification emails linked to CMS ticket details using the database ID. CMS DataTable and board links also used internal IDs.

## What I Tried

Traced the ticket link generation through the API email builder and CMS support ticket routes, table links, board cards, and status update calls.

## Root Cause

The system already had `ticket_number`, but API and CMS routing still treated the internal `id` as the route key.

## Fix

Changed API admin notification links to:

```text
/support-tickets/{ticketNumber}
```

Changed CMS support ticket model binding to use:

```php
public function getRouteKeyName(): string
{
    return 'ticket_number';
}
```

Updated CMS DataTable links, board cards, and status update URLs to use encoded ticket numbers.

Ticket numbers were also shortened to:

```text
TKT-######
```

## Verification

Added API test coverage for the ticket number format.

Checked CMS documentation so ticket detail, reply, status, and assign routes are documented as binding by `ticket_number`.

## Lesson

Do not expose database IDs in customer or admin URLs when a public reference number exists. Still keep authorization checks in place.

## Impact

Made support ticket links cleaner and reduced unnecessary exposure of internal IDs across email, CMS, and support workflows.

## Resume / Interview Bullet

Reduced support ticket IDOR risk by replacing internal ID-based links with public ticket-number routing across API notifications and CMS workflows.

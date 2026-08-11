---
date: 2026-08-04
type: FEATURE
tech: [Laravel, Next.js, CMS]
repos: [api.sarasavi.lk, cms.sarasavi.lk, www.sarasavi.lk]
tags: [branch-network, whatsapp, contact-hub, email, sms]
status: final
---

# Branch WhatsApp Was Added And Contact Details Were Centralized

## Problem

Branch WhatsApp numbers were not managed clearly as part of branch contacts. Customer-facing emails, SMS, print templates, and pages also had scattered phone numbers and email addresses.

## Context / Reproduction

Contact details appeared in several places:

- branch network page
- order emails
- dispatch SMS
- printed order and invoice templates
- stock and request-book notification emails

## What I Tried

Checked the CMS branch fields, API branch response, Web branch network page, email templates, SMS helper, and printed order templates.

## Root Cause

Contact details were duplicated across templates and messages instead of pointing customers to one maintained contact hub.

Branch WhatsApp was also missing as a separate CMS/API/Web field.

## Fix

Added WhatsApp as a separate branch contact field in CMS, API, and Web.

Updated customer-facing contact copy to point to:

```text
/branch-network
```

Kept branch-specific phone and WhatsApp details on the branch network page.

## Verification

Developer notes were updated across repos to document that `/branch-network` is the contact-directory hub and that WhatsApp stays separate from `telephone` and `phone`.

## Lesson

Keep public contact details in one maintained place. Templates should link to the contact hub instead of copying phone numbers and emails everywhere.

## Impact

Reduced stale contact details across customer communications and made branch WhatsApp details CMS-manageable.

## Resume / Interview Bullet

Centralized customer contact routing by adding CMS-managed branch WhatsApp data and updating Web, email, SMS, and print outputs to use the branch network contact hub.

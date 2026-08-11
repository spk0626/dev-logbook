---
date: 2026-08-10
type: BUG
tech: [Laravel, Next.js, Playwright]
repos: [api.sarasavi.lk, www.sarasavi.lk]
tags: [support-tickets, categories, api, e2e, seeding]
status: final
---

# Support Ticket Category Dropdown Was Empty And Blocked Ticket Creation

## Problem

The support ticket form category dropdown only showed the default "Select a category" option. Customers could not choose a ticket category, so ticket creation could not work correctly.

## Context / Reproduction

The storefront reads categories from:

```text
GET /api/support-ticket-categories
```

When the API did not return active category rows, the form had no real options.

## What I Tried

Checked the frontend form first, then traced the dropdown back to the API category endpoint and the default category data.

## Root Cause

The default support ticket categories were treated like normal seed data. Production needed those records available after deployment without depending on someone manually running a seeder.

## Fix

Moved the default 20 support ticket categories into a deploy-backed migration and kept the seeder available for local reseeding.

Added coverage:

```text
api.sarasavi.lk/tests/Feature/SupportTicketCategoryTest.php
www.sarasavi.lk/tests/e2e/support-ticket.spec.js
```

## Verification

API test checks that `GET /api/support-ticket-categories` returns active categories in sort order.

Web Playwright test mocks the category API response and confirms the selected `category_id` is submitted with the support ticket.

## Lesson

If a production form depends on default records, deploy those records with the app and test the API that serves them.

## Impact

Reduced the risk of customers being blocked from creating support tickets because category data was missing.

## Resume / Interview Bullet

Fixed support ticket category reliability by moving required defaults into deploy-backed data and adding API and Playwright coverage for the customer ticket flow.

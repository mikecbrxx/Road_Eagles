# Road Eagles MC Thailand — Club Management App

**Current release: v3.71**

A private progressive web app (PWA) for managing Road Eagles MC Thailand club operations, finances, merchandise, events, suppliers and membership.

## Overview

The application is a single-file HTML app. Club data is stored in Supabase and synchronised across devices. It runs in a modern mobile or desktop browser and can be installed as a PWA.

## v3.71 Release Notes

This release addresses responsible-person selection in merchandise sales and improves the warning shown when opening the payment history for a completed order settled through multiple payments.

- **Default responsible person:** the currently logged-in application user is selected by default.
- **Alternative responsible person:** other available club members remain selectable.
- **Pricing remains independent:** the Member/Visitor customer-pricing selection continues to determine the applicable price and member-only product eligibility. The responsible person does not determine the customer's pricing category.
- **Completed multi-payment orders:** the warning shown when opening the associated ledger/recent-transaction entry is clearer about the completed order and the available tab action.

Complete the club's normal regression checks after installing the update, particularly sales, part-payments, order completion, stock and ledger behaviour.

## Main Features

- **Dashboard** — account balances, month opening figures, stock alerts and tab net position.
- **Ledger** — income and expense tracking across Club and Charity accounts.
- **Members** — member records, role assignment and annual fee tracking.
- **Merchandise** — product catalogue, Member/Visitor pricing, stock levels and colour/size variants.
- **Tab Ledger** — open merchandise tabs, unpaid transactions and outstanding purchase-order balances.
- **Purchase Orders** — multi-line orders, production cost estimates, payments, assembly groups/BOM, event tagging and administrative status override.
- **Events** — event management and income/expense tracking. Entries tagged to an Open event are tracked but excluded from Club/Charity account totals until the event is Closed.
- **Suppliers** — supplier records.
- **Reports** — monthly and annual statements, club meeting report, membership, merchandise sales, stock, donations, tab debts, purchase ledger, cost of goods, analysis and audit trail.
- **Categories & Configuration** — income/expense categories, product categories, event types, sizes, colours, roles and assembly groups.
- **Backup & Restore** — JSON export/import, configurable filename prefix and backup reminders.

## Access and Roles

| Role | Access |
|------|--------|
| Admin | Full access, including settings, deletion, audit trail, backup and PO status override |
| Officer | Standard operational access |
| Title Only | Display role; no application login |

Application access is controlled by a 4-digit PIN. User access is managed by an Admin in **More → Settings**.

Application login users and club member records are separate data concepts. The v3.71 responsible-person default uses the logged-in application user; it does not change the customer's Member/Visitor pricing selection.

## Getting Started

### New Device Setup

1. Open the app URL in Chrome or another supported browser.
2. The app synchronises from Supabase before the PIN screen appears.
3. Enter your 4-digit PIN.
4. Follow the normal club device and access procedures.

### Updating to a New Version

1. In Chrome, open **Settings → Privacy → Clear browsing data**.
2. Select **Cached images and files only**.
3. Reload the app page so the new version is downloaded.
4. **Do not clear site data or cookies.** These may contain locally stored application data or credentials.

If the app does not show the new version, close the app/browser tab completely and reopen it after clearing the cached images and files.

### If Login Is Blocked

If the app displays a connection error at startup, check the internet connection and use **Try Again**. Login is intentionally blocked when Supabase cannot be reached; this prevents stale local data from being used as though it were current.

## File Structure

```text
road-eagles-mc.html    — Live production application
remc-test.html         — Test application (separate Supabase database)
README.md              — Project and release notes
```

The production HTML filename must remain exactly **road-eagles-mc.html**.

## Technical Details

- **Storage:** Supabase PostgreSQL through its REST API (`club_data` table, `main` row).
- **Local cache:** Browser localStorage (`remct4` key).
- **Sync:** Data is pushed to Supabase on save and fetched from Supabase when the app opens.
- **Offline behaviour:** Login is blocked if Supabase cannot be reached; this is intentional.
- **Application format:** Single-file HTML/PWA, with supporting manifest and icon assets in the deployment.

## Test System

The test app (`remc-test.html`) uses a separate Supabase project and separate localStorage keys. Live and test data are intended to remain independent.

To initialise the test database, run this in the test Supabase project's SQL editor:

```sql
INSERT INTO club_data (id, data, updated_at)
VALUES ('test_main', '{}'::jsonb, now())
ON CONFLICT (id) DO NOTHING;
```

Use the test system for controlled checks before making changes to live club data.

## Backup and Restore

- Admin only: **More → Settings → Backup & Restore**.
- Configure the filename prefix (default: `RE-Backup`).
- Backups are JSON files named using the configured prefix and a date/time stamp.
- Set the reminder interval in days; `0` disables reminders.
- Use **Save & Sync** to synchronise settings across devices.
- Restore accepts a valid backup JSON file regardless of its filename.

Keep a current backup before installing a new release or carrying out administrative data changes.

## Important Business Rules

- **Membership fees:** the full annual fee applies regardless of the member's join date; fees are not prorated.
- **Assembly-group purchase orders:** receiving individual components does not add finished-product stock. Stock is added through the Assemble step once the components are available.
- **Purchase-order balances:** a PO with an outstanding balance after stock receipt remains visible as partially received/owing until fully paid.
- **Tab deletion:** deleting an unpaid tab entry restores its stock. A tab is unpaid by definition, so deleting it does not create or reverse a financial transaction.
- **Donations:** Club and Charity donations are reported separately, with confirmed and pledged/pending amounts distinguished.
- **Open events:** entries tagged to an Open event are excluded from normal Club/Charity account totals until the event is marked Closed. Debt-tracking views continue to show amounts owed regardless of event status.
- **Merchandise sales:** the customer's Member/Visitor pricing selection controls the applicable price and member-only product eligibility. The responsible person is a separate field.

## Version History

| Version | Notes |
|---------|-------|
| **v3.71** | Responsible-person default now selects the logged-in application user while retaining the option to choose another responsible person. Customer Member/Visitor pricing and member-only product eligibility remain independent. Clarified the warning shown when opening the payment history for a completed order settled through multiple payments. |
| v3.70 and earlier | Earlier releases include finance/reporting, event, purchase-order, stock, tab-ledger, backup and synchronisation improvements. Refer to the previous project release history and change records for details. |

---

*Road Eagles MC Thailand — Private use only*

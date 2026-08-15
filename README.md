# Road Eagles MC Thailand — Club Management App

A private progressive web app (PWA) for managing club operations, finances, merchandise, events and membership for Road Eagles MC Thailand.

## Overview

Single-file HTML application with no server-side dependencies. All data is stored in [Supabase](https://supabase.com) and synced across devices. The app runs in any modern mobile browser and can be installed as a PWA on Android or iOS.

## Features

- **Dashboard** — account balances with month opening figures, stock alerts, tab net position
- **Ledger** — full income/expense tracking across Club and Charity accounts, paid/unpaid toggle
- **Members** — member management, role assignment, annual fee tracking and reset (full fee regardless of join date)
- **Merchandise** — catalogue with member/visitor pricing, stock levels, colour/size variants
- **Tab Ledger** — open merch tabs (edit/delete with stock restore), unpaid transactions, PO outstanding balances
- **Purchase Orders** — full lifecycle with multi-line-item support, production cost estimates, payments, assembly groups (BOM), event tagging, status override for admin
- **Events** — event management with income/expense tracking; each event has an Open/Closed status — while Open, everything tagged to it (paid or unpaid) is tracked but held out of Club/Charity account totals until the event is marked Closed
- **Suppliers** — full supplier records
- **Reports**
  - Monthly Statement (month/year selector, opening/closing balances, category subtotals)
  - Annual Summary (opening/closing balances, category subtotals)
  - Club Meeting Report (by month, full year, or custom date range; opening/closing balances; period activity summary)
  - Membership, Merchandise Sales, Stock, Stock Value
  - Donations (split by Club and Charity account, and by confirmed vs pledged/pending)
  - Tab Debts, Purchase Ledger (with committed PO balances), Cost of Goods, Analysis (Actual / Committed / Combined views), Audit Trail
  - Monthly Statement, Annual Summary and Club Meeting Report each show a note for any pending event income/expenses excluded from their totals
  - All reports export to PDF with period label included
- **Categories & Config** — income/expense categories, product categories, event types, sizes, colours, roles, assembly groups
- **Backup & Restore** — JSON export/import with configurable filename prefix and reminder system

## Access & Roles

| Role | Access |
|------|--------|
| Admin | Full access including settings, delete, audit trail, backup, PO status override |
| Officer | Standard operational access |
| Title Only | Display role, no app login |

Access is controlled by 4-digit PIN. Managed by Admin in More → Settings.

## Getting Started

### New Device Setup

1. Open the app URL in Chrome or Brave
2. The app automatically syncs from Supabase before the PIN screen appears — no setup needed
3. Enter your 4-digit PIN
4. Credentials are hardcoded in the app as a fallback

### Updating to a New Version

1. In Chrome: Settings → Privacy → Clear browsing data → **Cached images and files only**
2. Reload the page — the new version loads from GitHub
3. Do **not** clear site data or cookies

> **Note:** Due to file size, uploading via the GitHub web interface may be slow. Use Git on desktop for faster uploads.

### If Login is Blocked

If the app shows a connection error on startup, check your internet connection and tap **↺ Try Again**. Login is intentionally blocked when Supabase cannot be reached — this prevents stale local data being used.

## File Structure

```
road-eagles-mc.html         — Live production app (committed without a "-final" suffix)
remc-test.html              — Test system (separate Supabase database)
README.md                   — This file
```

## Technical Details

- **Storage**: Supabase PostgreSQL via REST API (`club_data` table, `main` row)
- **Local cache**: Browser localStorage (`remct4` key)
- **Credentials**: Hardcoded in app JS as fallback, also stored in `remct_sync` localStorage key
- **Sync**: On every save, data pushes to Supabase. On app open, data pulls from Supabase before PIN entry is permitted
- **Offline**: App blocks login when Supabase is unreachable — intentional by design

## Test System

The test app (`remc-test.html`) connects to a separate Supabase project (`bgajjoog`) and uses separate localStorage keys (`remct4_test`, `remct_sync_test`). Credentials are hardcoded and locked — live and test data are completely independent.

To initialise the test database, run in the test project SQL editor:
```sql
INSERT INTO club_data (id, data, updated_at)
VALUES ('test_main', '{}'::jsonb, now())
ON CONFLICT (id) DO NOTHING;
```

## Supabase Maintenance

Run annually on both live and test projects (before October 2026):

```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON public.club_data TO anon;
ALTER TABLE public.club_data ENABLE ROW LEVEL SECURITY;
SELECT * FROM pg_policies WHERE tablename = 'club_data';
```

## Backup

- Admin only: More → Settings → Backup & Restore
- Configure filename prefix (default: `RE-Backup`) — saved as `<prefix>-YYYYMMDD-HH:MM:SS.json`
- Set reminder interval in days (0 = disabled). Tap **Save & Sync** to push both settings to all devices
- Restore accepts any valid backup JSON file regardless of filename
- Opening Balances (More → Settings) are hidden by default and require re-entering your PIN to view or edit, since they underpin every balance and report in the app

## Purchase Orders

POs support multiple stock line items per order — useful for ordering multiple shirt sizes or product variants from one supplier in a single PO.

- Add stock items with **+ Add Stock Item** — select product, variant, size, colour and quantity
- Production costs are used to calculate an **estimated total** which can be applied with one tap
- Full lifecycle: Draft → Ordered → Part Paid → Part Rcvd → Received
- Assembly Groups (BOM) link multiple POs to one finished product — stock only added on assembly, not on individual component receipt
- Optionally tag a PO to an event

## Events & Account Totals

Each event has an **Open / Closed** status, set on the event modal (defaults to Open for new events).

- While an event is **Open**, everything tagged to it — ledger transactions, purchase orders, and merchandise sales (paid or unpaid) — is fully tracked and visible in the Event Report and Home screen event card, but is deliberately **excluded from Club/Charity account totals** (Dashboard balance, Monthly Statement, Annual Summary, Club Meeting Report, Analysis Report).
- Marking an event **Closed** brings everything tagged to it into the normal account totals immediately. Re-opening reverses this.
- A blue note ("Pending event entries excluded…") appears on the Dashboard and on Monthly/Annual/Meeting reports whenever there are open-event amounts being held back, so the gap between what's shown and what's confirmed is never silent.
- Debt-tracking views — **Tab Net Position**, **Tab Debts Report**, and the **Purchase Ledger**'s committed PO balances — intentionally do **not** apply this exclusion, since they track real amounts still owed regardless of event status.
- Merchandise sales tagged to an event show a snippet of the sale's **Notes** field in place of the usual member/customer tag (e.g. "Table – Standard ×1 — John & Sarah") — set Notes to whoever the sale is actually for. Non-event sales are unaffected.

## Key Business Rules

- **Membership fees**: Full annual fee applies to all members regardless of join date (no pro-rata)
- **Assembly group POs**: Stock is NOT added when individual components are received — only added via the Assemble step once all components arrive
- **PO status**: A PO with outstanding balance after stock receipt shows as "Part Rcvd" and remains visible in the Tab Ledger until fully paid
- **Tab deletion**: Deleting a tab entry restores stock levels. No financial transaction is created or reversed (tabs are unpaid by definition)
- **Donations**: Split by account — Club donations and Charity donations are reported separately, and each is further split into confirmed (paid, not tied to an open event) vs pledged/pending
- **Events**: See "Events & Account Totals" above — open-event amounts are excluded from account totals regardless of paid status, until the event is closed

## Version History

| Version | Notes |
|---------|-------|
| v3.48 | Batch fix: Annual Summary and Club Meeting Report gained pending-event banners and correct exclusions; Donation Report rebuilt to split confirmed vs pledged/pending; Analysis Report fixed so paid transactions tied to an open event no longer vanish from all views; Settings screen cleanup (removed redundant Categories/Stock/Suppliers shortcuts, Opening Balances hidden behind PIN re-entry) |
| v3.47 | Event-linked merchandise sales show a Notes snippet instead of the member tag; sale titles auto-regenerate if a sale's event link is changed later |
| v3.42–v3.46 | Events gained an Open/Closed status; open-event income/expenses (paid or unpaid) excluded from account totals until closed; fixed a stale-cached top-level script bug that could silently break app initialisation; fixed unsettled tab sales not counting toward event totals; fixed a lost event-tag bug on unpaid sales |
| v3.34–v3.41 | "isConfirmed()" helper introduced to centralise the paid/unpaid check used across balances and reports (pure refactor, no behaviour change); Purchase Orders gained an event field; fixed "View PO" button failing due to a temporal-dead-zone bug; added a dedicated "Purchase Orders" entry to the More menu |
| v3.33 | Pending event transactions excluded from Monthly Statement, Annual Summary and Meeting Report; dashboard indicator for pending event amounts; resetStockKey function fix |
| v3.29–v3.32 | Event field added throughout (transactions, sales, POs, tab settlement); Event Report with itemised income/expense; dashboard event overview block |
| v3.25 | Multi-line-item POs, production cost estimates, baseline v24 |
| v3.22 | Meeting report activity summary grouped by product |
| v3.19 | Remove pro-rata fees, PDF period label fix, baseline v21 |
| v3.17 | Delete tab entries with stock restore |
| v3.14 | Assembly group POs no longer add stock on receipt |
| v3.10 | Admin PO status override, part_received status |
| v3.08 | PO variant stored by ID not index, migration for existing POs |
| v3.00 | Assembly groups config, dynamic stock tabs, tab ledger edit, report opening/closing balances |
| v2.93 | Startup fetch gate, hardcoded credentials, login blocked offline |
| v2.88 | Data loss fix — manualSync only pushes on empty remote |
| v2.76 | Roles system with app access flags |

---

*Road Eagles MC Thailand — Private use only*

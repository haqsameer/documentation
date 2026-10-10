# Business OS UAT Data Seeder

An in-app, Owner/Admin-only tool that populates realistic, connected **UAT test data** into the **current
shop** so the deployed application can be exercised manually without asking anyone to run SQL.

## Where
Admin → route **`/admin/uat-data`** ("Business OS UAT Data"). Buttons: **Seed Full UAT Data**, **Remove UAT
Data**, and a live **seed status** (Not seeded / Partially seeded / Seeded).

## How to run
1. Sign in as the shop **owner or admin**.
2. Open `/admin/uat-data`.
3. Click **Seed Full UAT Data** and confirm. A per-module result list is shown (created / reused, with each
   module marked **ready** for UAT, or **manual** for posted/settled transactions).
4. To clean up, click **Remove UAT Data** and confirm.

## Security
- Runs entirely **server-side** (`src/lib/uat-seed.server.ts`) via the service-role client — credentials are
  never exposed to the browser.
- Every server function (`src/lib/uat-seed.functions.ts`) requires an authenticated user and
  **`requireRole(ADMINS)`** (owner/admin) on the target shop, so a user can only seed a shop they administer
  (no cross-tenant access). `shopId` is bound to the caller by that role check.
- Not callable anonymously.

## UAT naming / idempotency
- Everything uses the **`UAT-`** prefix with a deterministic natural key (slug / code / sku / name).
- Running Seed twice **reuses** existing rows instead of duplicating (idempotent).
- The deterministic identity is centralised in `src/lib/uat-seed-spec.ts` (unit-tested in
  `uat-seed-spec.test.ts`).

## What Seed creates (auto)
> **New-tenant bootstrapper.** Seed works on a **brand-new, near-empty shop** created from any account — it
> does not assume Trendies or any pre-configured merchant. A shop created through onboarding already gets a
> default Chart of Accounts, branch/register/location, storefront and settings via `provisionShop`; the seeder
> **reuses** those and additionally **enables every module** and **guarantees the accounting foundation** so no
> seeded page is empty or hidden. All IDs are resolved dynamically from the authenticated shop — nothing is
> hard-coded.

- **Modules** — enables all platform modules for the shop (`retail_pos`, `restaurant_pos`,
  `restaurant_tables`, `restaurant_kitchen`, `ecommerce`, `inventory`, `purchasing`, `accounting`,
  `employees`, `payroll`, `crm`, `loyalty`, `marketing`) via the authoritative `updateShopModules` merge, so a
  fresh `retail` shop can still show Restaurant/CRM/Marketing pages. Business type and other flags are preserved.
- **Foundation** — `UAT-Branch`, `UAT-Register`, `UAT-Location`, departments (`UAT-Management/Sales/Finance/HR/
  Kitchen/Operations`), cost centers (`UAT-Retail/Restaurant/Administration`).
- **HR** — designations + 6 employees (`UAT-Employee-…`) with department, designation and manager links.
- **Workforce** — 3 shifts (Day/Evening/Night, incl. overnight), 2 leave types (Annual/Sick), 1 holiday.
- **Retail** — `UAT-Retail` category + 3 products (`UAT-Water Bottle` healthy stock, `UAT-Coffee Mug` low stock,
  `UAT-TShirt` zero stock) with variant, sale price, cost and inventory.
- **Restaurant** — `UAT-Floor`, `UAT-Table-1..4` (each with a QR token so `/r/$token` works), `UAT-Kitchen
  Station`; raw ingredients with mixed stock units (`UAT-Bun` pcs, `UAT-Patty` **kg**, `UAT-Sauce` ml,
  `UAT-Cheese` pcs, `UAT-Fries Raw` kg) + stock; menu products (`UAT-Burger`, `UAT-Cheese Burger`, `UAT-Fries`,
  `UAT-Soft Drink`) with recipes (burger patty is authored in **grams** vs kg stock to exercise UOM
  conversion); a **required** single-select modifier group (`UAT-Cheese Option`: No/Single/Double) + an
  **optional** multi-select group (`UAT-Extras`) assigned to the burgers; and `restaurant_settings` set to
  public ordering + pickup + delivery + table QR ON with `menu_availability_mode = recipe_inventory` and a
  public ordering token (existing settings columns are updated in place — unrelated merchant settings are not
  overwritten).
- **CRM** — 5 customers (`UAT-Customer-…`), plus **tags** (`UAT-VIP/New/Restaurant/Retail/Dormant`) with
  assignments, staff **notes**, an **interaction** timeline entry per tagged customer, and varied
  **marketing-consent / suppression** states (opt-in / opt-out / suppressed — UAT customers only).
- **Segments/Audiences** — 5 dynamic `customer_segments` (`UAT-VIP/New/Dormant/Restaurant/High-Value-Customers`)
  + one static `customer_audience` (`UAT-VIP-Audience`) with members.
- **Payroll (config only)** — 5 `pay_components` (`UAT-BASIC/HRA/TRANSPORT/OT/ATT-DED`), `payroll_settings`
  defaults (inserted only if absent), one **open** `payroll_periods` for the current month, and a
  `employee_salary_structures` + components per UAT employee. **Payroll RUNS/posting stay manual.**
- **Attendance/Leave** — a few recent `attendance` days per employee (present/late) + `leave_requests` in
  **pending / approved / rejected** states.
- **Accounting (config + drafts)** — **ensures the default Chart of Accounts exists** (reusing the authoritative
  `ensureChartOfAccounts` in `shop.server.ts` — inserts only missing accounts, never duplicates), one **open**
  `fiscal_years` (`UAT-Fiscal Year`, set current only if the shop has no current year) + current-month
  `accounting_periods`, and 4 **DRAFT** `vouchers` (`UAT-JV/PV/RV/CONTRA-001`) with balanced `voucher_lines`
  built from the COA. **Vouchers are never posted** (status stays `draft`).
- **Bank Reconciliation** — one `bank_accounts` (`UAT-Bank-Account`, linked to the Bank GL account 1100) + a
  **DRAFT** `bank_reconciliations` session with unmatched `bank_statement_lines`. Matching/posting stays manual.
- **Budgets** — one **DRAFT** `budgets` (`UAT-Current-Month-Budget`) with `budget_lines` (approval stays manual).
- **Loans/Advances** — one salary advance (`UAT-ADV-001`, kind `advance`); **repayment posting stays manual**.
- **Loyalty** — a `loyalty_programs` row (reuses an existing program if present), 3 `loyalty_tiers`
  (`UAT-Bronze/Silver/Gold`) and a `loyalty_rewards`. The points **ledger** only accrues from real sales (manual).
- **Promotions** — 2 `promotions` (`UAT-10PERCENT` %, `UAT-FLAT20` fixed) each with a redeemable `coupons` code.
- **Marketing** — 3 **DRAFT** `marketing_campaigns` linked to segments, each with **queued** `campaign_recipients`
  (status `queued` so nothing is sent). **Sending stays manual — never auto-dispatched.**
- **Recruitment** — 2 `job_openings` (`UAT-Chef/Cashier-Vacancy`), 3 `candidates`, and `job_applications`
  spread across pipeline stages (applied/interview/offer).
- **Performance** — one **active** `performance_cycles` (`UAT-Performance-Cycle`) + `performance_goals` for
  several employees (reviews/ratings stay manual).
- **Purchasing** — 2 suppliers (`UAT-Supplier-Foods`, `UAT-Supplier-General`) + one **DRAFT** `purchase_orders`
  (`UAT-PO-001`) with lines against UAT ingredient variants. **Receiving/GRN/posting stays manual.**
- **AI** — a **non-secret** `ai_settings` row (provider `lovable`, default model), inserted only if absent.
  **Provider secrets and usage/tool-run history are never seeded.**

## What is intentionally left for MANUAL UAT (posted/settled transactions only)
To avoid producing **half-written or corrupt financial/inventory state**, only genuinely
**transaction-derived / posted** records must go through authoritative server flows — the tool reports these
as "manual":
- **Posted journals / voucher posting**, **settled sales**, **payroll RUNS**, **GRN/receiving**, and **AR/AP**
  (which arise from settled sales/purchases & posted vouchers). Config, master and **draft** data for every one
  of these areas **is** now seeded above — only the final posting/settlement step is manual.
- Restaurant **orders/KOT/settlement** are also left manual — settle through the existing Restaurant POS so
  `record_pos_sale` + reservation + accounting stay authoritative. The seeded floor/tables/menu/recipes/
  modifiers/stock are exactly what those manual flows need.

## Danger Zone — destructive tenant reset (Owner-only)
Beyond the safe UAT cleanup above, `/admin/uat-data` has an **Owner-only Danger Zone** with two separate
destructive actions. Both require typing the exact store name (re-validated **server-side**), run entirely via
the service-role client, and are strictly **tenant-scoped** — a caller can only ever wipe a shop they own.

**How it deletes (DB):** every shop-scoped table references `public.shops … on delete cascade` (verified across
the whole schema — 173 FKs). So the wipe deletes the **`shops` row** and Postgres cascades the entire tenant
tree atomically, in correct FK order. This can never drift out of sync with the schema (no hand-maintained
delete list).

**How it deletes (storage):** the three tenant buckets — `product-images`, `hr-documents`, `payment-proofs` —
all use a `<shop_id>/…` path prefix (enforced by RLS `split_part(name,'/',1)::uuid`). The wipe recursively
lists and removes only the objects under the shop's own prefix — never a shared bucket wipe, never another
tenant's files. Prefix deletion also removes any orphaned objects, so none remain.

| Action | Business data & files | Store row | Memberships | Login (Supabase Auth) |
|--------|----------------------|-----------|-------------|-----------------------|
| **Clear ALL Tenant Data** | Deleted (near-fresh empty tenant) | **Kept** (row re-inserted with same id) | **Kept** (re-inserted) | **Untouched** |
| **Delete Tenant Completely** | Deleted | **Removed** | Removed (this shop only, via cascade) | **Untouched** |

`Clear` captures the `shops` row + `shop_users`, deletes the shop (cascade), then re-inserts the same shop row
and memberships → identity preserved, all data gone, ready to **re-seed**. `Delete Completely` removes the shop
row and does not restore it. **Neither deletes the Supabase Auth account** (a user may own other shops); account
deletion is a separate operation. Each run returns explicit counts: database rows deleted, storage files
deleted, storage failures, buckets affected, orphan files, and **residual** tenant rows/files (verification).

## UAT lifecycle
```
Fresh account → Create shop → Seed / Repair Full UAT Data → Test entire Business OS
      ├─ Clear ALL Tenant Data → Seed again → …   (repeatable clean-room UAT cycles; shop + login kept)
      └─ Delete Tenant Completely                 (store removed; login kept, may own other stores)
```
- **Remove UAT Data** (safe): removes only deterministic `UAT-` records; keeps real data & the shop; respects
  posted/settled history.
- **Clear ALL Tenant Data** (destructive): empties the tenant, keeps the shop + owner, re-seed to refill.
- **Delete Tenant Completely** (destructive): removes the shop row too; login/account remains.
Re-running `Clear` after a successful clear is harmless (nothing left). `Delete Completely` on an
already-deleted shop fails cleanly as **"Tenant not found."** and never touches another tenant.

## Remove / cleanup (UAT-only)
`removeBusinessOsUatData` deletes **only** the exact deterministic UAT natural keys, **scoped to the shop**,
in FK-dependency order, and returns `{ removed, blocked }`. It **protects posted/settled financial state**:
before deleting it checks status and **skips + reports** (in `blocked`) any UAT record that has been advanced
beyond draft — a **posted/locked voucher**, a **non-draft purchase order**, a **non-draft budget**, or a
**loan with posted repayments** (whose owning employee is also protected from cascade). It never uses a broad
`LIKE 'UAT-%'` and never touches real merchant records. Shop-level config rows — `restaurant_settings`,
`loyalty_programs`, `ai_settings`, `payroll_settings` — are intentionally left in place.

## Files
- `src/lib/uat-seed-spec.ts` — deterministic UAT identity (shared, pure).
- `src/lib/uat-seed.server.ts` — server seeder (seed / remove / status).
- `src/lib/uat-seed.functions.ts` — Owner/Admin-gated server functions.
- `src/lib/tenant-admin-spec.ts` — pure tenant-wipe constants + typed-name confirmation (shared).
- `src/lib/tenant-admin.server.ts` — Clear / Delete tenant + storage wipe + residual verification (service-role).
- `src/lib/tenant-admin.functions.ts` — **Owner-only** server functions with server-side name confirmation.
- `src/routes/_authenticated/admin.uat-data.tsx` — the admin panel (Seed / Remove + Danger Zone).
- `src/lib/uat-seed-spec.test.ts` — identity / idempotency / cleanup-targeting tests.
- `src/lib/tenant-admin-spec.test.ts` — bucket scoping / verification-set / confirmation-name tests.

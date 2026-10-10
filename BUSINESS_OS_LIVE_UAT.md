# Business OS — Live UAT — FINAL HANDOFF (authoritative)

> **This top section is the single source of truth for production state (as of 2026-09-14).**
> Everything below the "HISTORICAL / SUPERSEDED UAT LOG" divider is a chronological evidence log kept for
> traceability. Where an earlier entry says P1/P0/FAIL/CONDITIONAL-GO, it has since been **resolved and
> superseded** — trust this section, not the historical entries.

**Environment:** deployed app `https://goretailos.com` · Supabase project `tgaibydudmsslfxyzipz`. Verified live via the integrated browser UI + authenticated Supabase REST ground-truth reads. All disposable UAT tenants have been deleted; the owner auth account and unrelated shops are intact.

## CURRENT PRODUCTION STATUS

**PRODUCTION READINESS: GO** (for the tested surface)

| Area | Status |
|------|--------|
| **P0 OPEN** | **0** |
| **P1 OPEN** | **0** |
| Migration-application audit | **PASS** |
| Bank-transfer proof verification | **PASS** |
| Tenant isolation | **PASS** |
| Advance / Loan **Issue & Post** | **PASS** |
| Seed idempotency | **PASS** |
| Disposable tenant lifecycle / deletion | **PASS** |
| TypeScript (`tsc --noEmit`) | **0 errors** |
| Vitest | **261 / 261 PASS** |
| Production build (`vite build`) | **PASS** |
| **Concurrency / last-portion oversell** | **NOT IN CURRENT UAT SCOPE** — intentionally skipped by user decision (not a blocker, not a defect; non-concurrent oversell-at-sale protection is separately evidenced) |
| **Roles / permissions** | **NOT COMPLETED** — environment-gated: no restricted-role member could be seated to exercise denials. **No known defect** (RLS + `has_shop_role` guards present in code; not asserted PASS by inspection alone). |

### Evidence pointers (live-verified)
- **Migration audit:** 18 restaurant/proof tables, 22 additive columns, 9 critical RPCs present on live; only `increment_coupon_usage` absent (never migrated; unwired promotions — see follow-up 6).
- **Bank-transfer proof:** merchant "Verify & mark paid" posts a balanced **Dr 1100 Bank / Cr 1150 AR** payment journal, idempotent on reload; upload never auto-verifies.
- **Tenant isolation:** a genuine non-member second account was denied all cross-tenant reads/writes/storage; only the intentional public storefront catalog is visible (no cost/supplier fields).
- **Advance/Loan Issue & Post:** loan → `active`, balanced **Dr 1150 Accounts Receivable / Cr 1000 Cash $20,000**, 4×$5,000 schedule, single journal on reload (no double-post).
- **Tenant deletion:** both disposable shops removed with **0 residual rows/files**, storage objects cleaned, **auth account + other tenants untouched**.

## NON-BLOCKING FOLLOW-UPS (no code work performed for these in this cycle)
1. **Complete Supabase type regeneration** from the fully-migrated live schema (it currently still omits `attendance_records`, `leave_balances`, `attendance_settings`, `product_modifiers`, `recipe_ingredients`) **before removing the remaining DB compatibility shims** (`rdb`/`cdb`/`hdb`/`fdb`/`edb`/`rsupa`). These shims are intentionally retained until then; the typed client also currently hits `TS2589` on the missing-table paths.
2. **Expose Advance Receivable / loan-funding account configuration in Payroll Settings** (today only the UAT seeder sets `payroll_settings.advance_receivable_account_id` / `loan_receivable_account_id` / `loan_funding_account_id`; there is no admin UI field, so a real merchant tenant can't configure them without the seeder).
3. **Restaurant table-transfer UI** — server fn `transferTable` exists but has no UI caller.
4. **Restaurant split / partial-payment UI** — settle dialog is single-method full payment; no allocation UI.
5. **Ecommerce checkout coupon-entry UI** — coupons/promotions exist in data but there is no customer coupon field at checkout.
6. **`increment_coupon_usage`** — absent from all migrations; only called on an error-swallowed promotions path. Create/wire it when promotions are fully implemented.

---

# HISTORICAL / SUPERSEDED UAT LOG

> ⚠️ **Everything from here down is historical evidence, kept in chronological order.** Any P0/P1/FAIL/
> CONDITIONAL-GO/"pending deploy" wording below reflects the state **at the time of that entry** and has since
> been **superseded** by the CURRENT PRODUCTION STATUS above. Do not treat the historical verdicts as current.

## Part 1 — Early partial UAT (tenant `test2`)

**Environment:** deployed app `https://goretailos.com` · tenant **test2** (slug `adsas`, shop `6646e8da-72d3-4513-8e42-771528ae2316`) · signed in as **owner** · Supabase project `tgaibydudmsslfxyzipz`.
**Method:** real UI driven through the integrated browser + authenticated Supabase REST reads for ground-truth verification. Findings below are observed live, not inferred.

> _(Superseded)_ This was a **partial** live UAT. It established seed integrity, verified broad module rendering, found and
> fixed one real **P1**, and identified a **P2** data-placement issue that blocks the transactional restaurant/POS
> phases. Full transactional flows and the P1 live-retest require a redeploy (see "Remaining actions").

## Results table
| ID | Module / Scenario | Result | Evidence |
|----|-------------------|--------|----------|
| 1 | Seed sanity | **PASS** | Departments (UAT-MGMT/SALES/FIN/HR/KIT/OPS), designations, customers, products, suppliers, loans, restaurant floor+tables all render live |
| 1 | Modules enabled | **PASS** | Seed "Modules → Enabled all UAT modules"; Restaurant/CRM/Marketing pages reachable |
| 2 | Employees directory | **PASS (P1 fixed + retested live)** | After deploy: directory lists all 6 UAT employees with departments, designations, branch, status, and manager ("Ali UAT-Khan") resolving |
| 3 | Attendance list | **FAIL — P2** | "No attendance yet" — page date-window default vs seeded recent dates (data exists) |
| 6 | Accounting overview | **PASS (render)** | Loads; "Ledger is balanced: debits $0.00 = credits $0.00" (correct — vouchers are unposted drafts) |
| 11 | Retail POS sale | **BLOCKED — P2 stock placement** | Seeded stock is at non-default "UAT-Location"; POS uses default "Main Store" (0 stock). See P2 |
| 13 | Customers / CRM | **PASS** | All 5 UAT customers render with phone/consent |
| 17 | Restaurant foundation | **PASS** | Dedicated POS shell (not admin shell; "Exit to admin", fullscreen); UAT-Table-1..4 on UAT-Floor, all "available" |
| 20-21 | Reservation / settlement | **BLOCKED — P2 stock placement** | Recipe dishes "Unavailable" because ingredient stock is at non-default location |
| 23 | Restaurant availability (customer) | **PASS** | Storefront shows dish-level availability only (Soft Drink orderable; recipe dishes "Unavailable"); no ingredient internals exposed |
| 30 | General ordering `/order/<token>` | **PASS (structure) + P1 found** | Branded customer site, menu, prices, cart, hours; recipe availability correct after P2 fix. Modifier sheet opens with required/optional groups + prices — BUT groups duplicated 4x (see P1 below) |
| 30 | Modifier customization sheet | **PASS (P1 #2 fixed + retested live)** | After deploy + Remove/Reseed: each group renders once; Single Cheese (+$10) -> $660 correctly |
| 39 | Loans / Advances | **PASS** | UAT-ADV-001 · Bilal UAT-Raza · Advance · $20,000.00 · Draft (proves employees exist + joinable) |
| 48 | Seed idempotency | **PASS (after ensure() fix)** | Module counters "0 created · N reused"; the null-match duplicate accumulation is fixed in `ensure()` |
| 4,5,7-10,12,14-16,18-19,22,24-29,31-38,40-47,49 | HR-detail, Payroll run, Purchasing GRN, POS sale, returns, voucher posting, loyalty earn, KOT/KDS, QR/kiosk submit, tenant isolation, storage, clear/delete, etc. | **NOT RUN / BLOCKED** | Require the P1 deploy, the P2 stock fix, a 2nd tenant (isolation), or are destructive (clear/delete). Not fabricated as PASS |

## P1 (FIXED in code, live-retest pending deploy) — Employee Directory returns empty
**Symptom:** `/admin/employees` shows "No employees"; searching "UAT" also returns nothing. Attendance and other
employee-list surfaces empty too.

**Proof employees exist:** the Loans page rendered `UAT-ADV-001 · Bilal UAT-Raza · Advance · $20,000.00` — i.e.
the loan joined to employee **UAT-EMP-BILAL** and resolved its name. Seed re-run also reported "HR: 0 created ·
12 reused".

**Root cause (confirmed via live REST):** the employees list embeds `departments(name)`. There are **two**
foreign keys between `employees` and `departments` (`employees.department_id -> departments` and
`departments.manager_employee_id -> employees`), so PostgREST cannot disambiguate and returns
**HTTP 300 `PGRST201`**. `listEmployees` therefore throws; the panel renders empty on any error (no error branch).
`designations(name)` and `branches(name)` embeds return 200 — only `departments(name)` was ambiguous.

Evidence:
```
GET employees?...&select=id,departments(name)      -> 300 PGRST201 (ambiguous: employees_department_id_fkey
                                                      vs departments_manager_employee_fk)
GET employees?...&select=id,departments!employees_department_id_fkey(name)  -> 200  (fix works)
```

**Fix (localized, low-risk):** qualify the embed by FK name `departments!employees_department_id_fkey(name)` in
the 3 affected employee queries:
- `src/lib/employees.server.ts` -> `listEmployees` (directory) and `getEmployee` (detail)
- `src/lib/hr-advanced.server.ts` -> `hrReports` (HR analytics/headcount)

Other `departments(name)` embeds (`designations.server.ts`, `job_openings`, `vouchers`) are single-FK and
unaffected — left unchanged. Validation after fix: **tsc 0, Vitest 258 passed, build success.**

**Retest status (DONE — PASS):** deployed and live-retested on goretailos.com. The Employee Directory now lists
all 6 UAT employees with departments/designations/branch/status and manager relationships resolving correctly.
Fix committed as `c8e881f` (rebased onto latest Lovable HEAD; tsc/vitest/build green). Also independently proven
against the live DB via REST (disambiguated embed returns 200 vs the broken 300).

## P2 (FIXED in code, pending deploy + re-seed) — seeded inventory placed at a non-default location
The seeder stocked all inventory (retail + ingredients) only at the **"UAT-Location"** (`is_default = false`),
while the general storefront/POS/settlement resolve the location via `defaultLocationId(shopId, branchId)`: the
general ordering site passes `branchId: null` (restaurant-public.server.ts:139), which resolves the shop's
**default "Main Store"** — where stock was 0, so recipe dishes showed **"Unavailable"** and POS/settlement would
see zero stock. (QR/table orders, being UAT-Branch-scoped, resolved UAT-Location and did have stock.)

**Fix:** `seedFoundation` now resolves the shop default location via the same authoritative
`defaultLocationId(shopId, null)` and `ensureProduct` seeds the quantity at **every** resolved stock location
(UAT-Location **and** the shop default), so every surface reads stock regardless of how the branch resolves.
Cleanup stays safe (inventory_balances cascade when UAT products are deleted). Validation: tsc 0, Vitest 258,
build success. **Retested live (PASS):** after deploy + one Seed/Repair re-run, the general ordering storefront
shows **0 "Unavailable"** dishes — UAT-Burger/Cheese-Burger/Fries all have enabled "Add" buttons.

## P1 #2 (FIXED + retested live — PASS) — modifier groups duplicated, price multiplied
On `/order/<token>`, opening UAT-Burger's customization sheet rendered the **UAT-Cheese Option** and
**UAT-Extras** groups **4 times each**, and selecting "UAT-Single Cheese (+$10)" changed the total from $650 to
**$690 (+$40)** — the adjustment was summed across the 4 duplicated groups.

**Root cause (confirmed via live REST):** the burger has **8** `product_modifier_groups` links for only 2 groups
(each linked 4x). The seeder's `ensure()` helper matched the existing link with `.eq("variant_id", null)`, but
`variant_id = NULL` is never true in SQL, so it never found the existing row and **inserted a fresh duplicate on
every seed run** (Seed/Repair was run ~4x during this UAT). This is simultaneously a **seed non-idempotency bug**
(silent duplicate accumulation for any null-matched column) and a **customer-facing price bug**.

**Fix (localized):** `ensure()` now uses `.is(col, null)` for null match values (and `.eq` otherwise), so
existing rows are found and no duplicate is inserted — idempotent for all call sites. Validation: tsc 0, Vitest
258, build success. **Cleanup of existing test2 data:** run **Remove UAT Data** then **Seed** after deploy —
Remove cascades the duplicate links away and the fixed seeder recreates exactly one link per group.

Note: the storefront price is display-side; the restaurant ordering backend recomputes modifier prices
server-authoritatively on submit, so this did not necessarily corrupt a submitted order total — but the
customer-visible price and the duplicated UI are a clear P1 for the ordering experience.

**Retest status (DONE — PASS):** deployed (`6396a99`), then ran **Remove UAT Data → Seed** on test2 to purge the
accumulated duplicate links and recreate clean data. Re-opening UAT-Burger's sheet now shows each modifier group
exactly once, and selecting "UAT-Single Cheese (+$10)" moves the total from $650 to **$660** (correct).

## Attendance empty (P2)
`/admin/attendance` shows "No attendance yet". Seeded attendance is for the last 3 days; the page's default
date window likely doesn't include them. Data exists; classified P2 (not investigated further this pass).

## Final summary
**LIVE UAT STATUS: CONDITIONAL — three defects found and ALL fixed + deployed + live-retested (PASS); the deeper transactional phases remain to be executed.**

- Passed (verified live): 12 — seed sanity, seed idempotency, module enablement, customers, products, loans, accounting render, restaurant foundation/dedicated shell, general ordering storefront, recipe availability, employee directory (post-fix), modifier customization sheet (post-fix).
- Failed P0: 0
- Failed P1: 2 — Employee Directory embed + storefront modifier duplication. **Both FIXED, deployed, and live-retested PASS.**
- Observed P2: 2 — inventory seeded to non-default location (**FIXED + retested**); attendance date-window default (report only).
- Not run / Blocked: transactional flows (payroll run, POS sale, purchasing GRN, restaurant KOT->settlement, returns, voucher posting, loyalty earn, QR/kiosk order submit), tenant isolation (needs 2nd account), concurrency/oversell, and clear/delete tenant (destructive — deferred to end).

### Fixes shipped this session (committed + pushed + deployed + live-retested)
1. **P1 — Employee Directory empty** (`c8e881f`): FK-qualified `departments!employees_department_id_fkey(name)` embed. Retest: all 6 employees list with departments + managers. **PASS.**
2. **P2 — seeded stock on non-default location** (`c8e881f`): seeder stocks every resolved location. Retest: storefront shows 0 "Unavailable" dishes. **PASS.**
3. **P1 — modifier groups duplicated / price 4x** (`6396a99`): `ensure()` uses `.is(col, null)` for null matches (idempotency). Retest (after Remove+Reseed): each group renders once, Single Cheese +$10 -> $660. **PASS.**

### Core workflows
- HR -> PAYROLL -> ACCOUNTING: **PARTIAL** (HR directory now works; payroll calc/post not executed)
- PURCHASING -> INVENTORY -> AP -> ACCOUNTING: **NOT RUN** (draft PO renders; GRN/posting not executed)
- RETAIL POS -> INVENTORY -> ACCOUNTING -> CRM: **NOT RUN** (stock now on default location — unblocked, not executed)
- RESTAURANT -> KOT -> RESERVATION -> SETTLEMENT -> INVENTORY -> ACCOUNTING: **NOT RUN** (availability + modifiers now correct — unblocked, not executed)
- CRM -> SEGMENT -> PROMOTION -> CAMPAIGN: **PARTIAL** (data renders; end-to-end not executed)
- RECRUITMENT -> EMPLOYEE: **NOT RUN**
- STOREFRONT -> ORDER -> PAYMENT PROOF: **PARTIAL** (storefront + modifier flow verified; order submit not executed)
- TENANT ISOLATION: **BLOCKED** (no second tenant/account available this session)
- SEED -> CLEAR -> RESEED: **PARTIAL** (Remove + Seed executed successfully; full Clear-tenant not run)
- SEED IDEMPOTENCY: **PASS**
- DELETE TENANT COMPLETELY: **NOT EXECUTED**

**PRODUCTION READINESS: CONDITIONAL GO** — the three defects found are fixed and live-verified, and the seeded tenant now renders correctly across modules. Remaining before a full GO: execute the end-to-end transactional flows (payroll, POS/restaurant settlement, purchasing GRN), the P0 tenant-isolation test (needs a 2nd account), and the destructive Clear/Delete-tenant phases.

### Remaining actions
1. Execute the transactional workflows now unblocked: Retail POS sale, Restaurant KOT->reservation->settlement, Payroll calc/post, Purchasing GRN, voucher posting — verifying inventory/accounting effects.
2. Provide a **second tenant/account** to run the P0 tenant-isolation test.
3. Run destructive Clear/Delete-tenant (Phases 46/49) at the very end on this disposable tenant.

## Automated validation (after each fix)
`tsc --noEmit` = 0 errors · Vitest = **258 passed / 20 files** · `vite build` = success. (Confirmed green after all three fixes; not a substitute for the live retests, which were performed on goretailos.com and all passed.)

---

# Transactional UAT (continued) — real end-to-end workflows with before/after evidence

All executed live on goretailos.com / tenant test2 / owner, verified against the live DB via authenticated REST.

## PHASE 1 — Payroll → Accounting: **PASS**
Run **UAT-Run-Sept** (period UAT-Payroll 2026-09-01): 6 employees, each Gross **$120,000** (base $50k + UAT-Basic $50k + HRA $15k + Transport $5k), deductions $0, Net $120k. Totals Gross/Net **$720,000**.
- Draft → Calculate → Approve → **Post** → journal **PAY-00001** (2026-09-30):
  - **Dr 6100 Salaries $720,000** (6 × $120k) / **Cr 2000 Accounts Payable $720,000** — balanced.
- **Payroll payment** (Ali, $120k via bank) → separate journal **SALPAY-00001**: **Dr 2000 AP $120,000 / Cr 1100 Bank $120,000** — settles payable, **no duplicate expense posting**.
- **P2 (report only) — seeder does not map payroll GL accounts.** Out of the box, posting failed with "Set the Employee Payable account…". Resolved via standard owner config (Payroll Settings → Salary expense 6100, Employee payable 2000) + Recalculate (earning-line expense account is snapshotted at calculate time, payroll.server.ts:562). Also a **cash account is not configured** so cash salary payment is blocked (used bank instead). These are seeder/config gaps, not app bugs — recommend the seeder map default payroll accounts.

## PHASE 2 — Accounting Vouchers (post + reversal): **PASS**
- **UAT-JV-001** (5000 COGS Dr $500 / 1000 Cash Cr $500): draft → Approve → **Posted**.
- **Reversal**: status → "reversed"; original lines intact; a **separate contra entry UAT-JV-001-REV** created (5000 COGS Cr $500 / 1000 Cash Dr $500). Ledger "balanced: debits = credits" throughout (Trial-Balance integrity holds).

## PHASE 4 — Purchasing → GRN → Inventory: **PASS** (AP via separate supplier-invoice step)
PO **UAT-PO-001** (draft, $18,320: Patty×20, Sauce×20, Bun×20) → Receive → **GRN-000001 posted** ("stock and costs updated").
- Inventory **+20 exactly** into Main Store (default): Patty **3→23**, Bun **40→60**, Sauce **2000→2020**. No double-stock.
- The GRN updates inventory + moving-average cost but posts **no GL journal** (by design — success text says "stock and costs updated"); AP/GL is created by the supplier-invoice step (Payables shows "Nothing outstanding" until then). Supplier-invoice → AP journal not exercised this pass.

## PHASE 6/10/11/22 — Restaurant dine-in → KOT → Reservation → Settlement → Inventory → Accounting: **PASS**
Order **RO-000001** on UAT-Table-1 (dine-in): added UAT-Burger (No Cheese) $650, modifier sheet showed **single** groups (dup-fix holds in POS too).
- **KOT sent → reservation created** in `restaurant_ingredient_reservations` at UAT-Location: Bun **1**, **Patty 0.15** (recipe 150 g → **0.15 kg**, UOM correct), Sauce **20** ml.
- **Settle $650 (cash) → inventory consumed exactly once** at UAT-Location: Patty **3→2.85** (−0.15 kg), Bun **40→39** (−1), Sauce **2000→1980** (−20). Movements recorded as `sale` (−1 / −0.15 / −20).
- **Reservation released** (0 remaining). **Table → available**. Order → settled.
- **Accounting** (memo "Sale", ref order): **Dr 1000 Cash $650 / Cr 4000 Sales Revenue $650** and **Dr 5000 COGS $170 / Cr 1200 Inventory $170** (Patty 0.15×$900=$135 + Bun 1×$15 + Sauce 20×$1). Balanced.
- Note: dine-in POS resolves the **branch** location (UAT-Location) while the general online storefront (`branchId null`) resolves **Main Store** — different stock pools (legitimate multi-location; both stocked by the seeder fix).

## Final summary (updated)
**LIVE UAT STATUS: CONDITIONAL GO** — all core financial & inventory workflows now execute correctly end-to-end with balanced accounting; three earlier defects fixed+deployed+retested; remaining phases are lower-risk operational flows, a 2nd-account security test, and destructive resets.

- Passed (verified live): 18 — seed sanity, idempotency, module enablement, customers, products, loans, employee directory (fixed), modifier sheet (fixed), general storefront + availability (fixed), **Payroll→Accounting**, **payroll payment (no double-post)**, **voucher post**, **voucher reversal**, **Purchasing→GRN→Inventory**, **ingredient reservation**, **restaurant settlement (consume-once + accounting)**, **UOM conversion (g↔kg, ml)**, restaurant dedicated shell.
- Failed P0: **0**
- Failed P1: 2 (Employee Directory embed, modifier duplication) — **both FIXED + deployed + retested PASS**.
- P2 (report only): (a) seeded stock at non-default location — **FIXED**; (b) attendance date-window default; (c) **seeder doesn't map payroll GL / cash accounts** (blocks out-of-the-box payroll posting until owner maps them); (d) base salary duplicated as a pay component (inflates gross in seed data).
- Not run / Blocked: Retail POS sale, retail return/refund, loyalty earn/redeem, promotions at checkout, QR/kiosk/online order submit, ecommerce COD + bank-transfer proof, recruitment/performance/budget transitions, roles/permissions enforcement, tenant isolation (needs 2nd account), oversell/concurrency, Clear/Reseed/Delete (destructive — end only).

### Core workflows (updated)
- HR → PAYROLL → ACCOUNTING: **PASS** (posted PAY-00001 balanced; payment SALPAY-00001 no double-post)
- PURCHASING → INVENTORY → AP → ACCOUNTING: **PASS (inventory)** / AP journal via supplier-invoice **NOT RUN**
- RETAIL POS → INVENTORY → ACCOUNTING → CRM: **NOT RUN**
- RESTAURANT → KOT → RESERVATION → SETTLEMENT → INVENTORY → ACCOUNTING: **PASS** (consume-once, UOM correct, balanced Sale+COGS)
- CRM → SEGMENT → PROMOTION → CAMPAIGN: **PARTIAL** (data renders; end-to-end not executed)
- STOREFRONT → ORDER → PAYMENT PROOF: **PARTIAL** (storefront + modifier verified; submit not executed)
- TENANT ISOLATION: **BLOCKED** (no second account this session)
- SEED → CLEAR → RESEED: **PARTIAL** (Remove + Seed executed; full Clear-tenant not run)
- SEED IDEMPOTENCY: **PASS**
- DELETE TENANT COMPLETELY: **NOT EXECUTED**

**PRODUCTION READINESS: CONDITIONAL GO.** The financial engine (payroll, vouchers, purchasing inventory, restaurant settlement with reservation + UOM) is verified correct and balanced on the live app. Before a full GO: run the remaining operational flows (Retail POS, returns, loyalty, promotions, online/QR/kiosk submit, ecommerce payment proof), enforce-test roles/permissions, run the P0 tenant-isolation test with a second account, and finally the destructive Clear/Delete-tenant phases. Recommended P2 seeder enhancement: map default payroll GL + cash accounts so payroll posting works out of the box.

---

# Transactional UAT — Part 2 (retail, purchasing AP, customer online ordering) + seeder P2 fixes

## Seeder P2 fixes (explicitly authorized) — shipped `5e3fb69`, tsc 0 / Vitest 261 / build OK
1. **Payroll account mapping** — `seedPayroll` now resolves COA by code and sets `payroll_settings.salary_expense_account_id` (6100), `employee_payable_account_id` (2000), `deduction_liability_account_id` (2100), backfilling only missing values; also seeds a `cash_accounts` row (UAT-Cash Drawer → GL 1000) for cash payroll payments. Goal: fresh Seed → Payroll Calculate → Approve → Post with **no manual Payroll Settings**. (To be proven live at Clear→Reseed once deployed.)
2. **Duplicated base salary removed** — structure attaches only allowance earnings (HRA + Transport) on top of `base_salary`; the redundant UAT-BASIC earning is no longer attached. Seeded gross = 50,000 + 15,000 + 5,000 = **70,000**. New spec tests assert no duplicate basic, gross=70,000, unique structure earning codes (idempotent re-seed).

## PHASE — Purchasing → AP → Accounting (completes the chain): **PASS**
Supplier invoice **INV-UAT-PO-001** created from the received PO, posted:
- **Dr 1200 Inventory $18,320 / Cr 2000 Accounts Payable $18,320**. AP before invoice $0 → after $18,320.
- Supplier **payment $18,320 (Bank transfer)** → **Dr 2000 AP $18,320 / Cr 1100 Bank $18,320**. AP after payment **$0** ("No outstanding balances").

## PHASE — Retail POS → Inventory → Accounting → CRM → Loyalty: **PASS**
Sale **ORD-000002** (receipt RCP-000002), register session opened, 2 × UAT-Water Bottle (UAT-SKU-WATER), customer **UAT-Customer-Ahmed**, $240 cash.
- Stock (Main Store default) **80 → 78** (−2 once). Accounting: **Cash Dr $240 / Sales Cr $240** + **COGS Dr $140 / Inventory Cr $140** (2 × $70).
- CRM: order links to the customer. **Loyalty earned 240 points** (`loyalty_ledger` earn/order; DB unique index prevents duplicate earning on retry).

## PHASE — Retail Return / Refund: **PASS**
Return **RET-000001** for 1 × Water Bottle, $120 (restock ON):
- Stock **78 → 79** (+1). Accounting reversal: **Sales Revenue Dr $120 / Cash Cr $120** + **Inventory Dr $70 / COGS Cr $70**.
- Original **ORD-000002 remains traceable** (payment status → "partial"); RET-000001 references it.

## PHASE — General Restaurant Pickup: **PASS**
`/order/<token>` → Soft Drink → Pickup + customer details → **RO-000002**: `order_type = takeaway`, `channel = online`, server-authoritative total **$180**, customer captured. It is a **restaurant_order**, NOT an ecommerce order.

## PHASE — General Restaurant Delivery: **PASS**
`/order/<token>` → Fries → Delivery + address → **RO-000003**: `order_type = delivery`, `channel = online`, total **$300**, `delivery_address` captured (no delivery fee configured → $0). A **restaurant_order**.

## Final summary (Part 2, updated)
**LIVE UAT STATUS: CONDITIONAL GO** — every core financial & inventory workflow executes correctly end-to-end with balanced accounting; two seeder P2 gaps fixed+shipped; remaining are operational flows, a 2nd-account security test, and destructive resets.

- P0 OPEN: **0**
- P1 OPEN: **0** (2 earlier P1s fixed+deployed+retested)
- P2 DEFERRED: attendance date-window default; (fixed this pass: payroll account mapping, base-salary duplication, stock location).
- BLOCKED: TENANT ISOLATION (no 2nd account); genuine OVERSELL/CONCURRENCY (no true concurrent sessions).

### Core workflows (final)
- HR → PAYROLL → ACCOUNTING: **PASS**
- PURCHASING → INVENTORY → AP → ACCOUNTING: **PASS** (GRN→invoice→AP→payment, all balanced)
- RETAIL POS → INVENTORY → ACCOUNTING → CRM: **PASS** (+ loyalty earn, + return/refund reversal)
- RESTAURANT → KOT → RESERVATION → SETTLEMENT → INVENTORY → ACCOUNTING: **PASS** (consume-once, UOM correct)
- RESTAURANT CUSTOMER ORDERING: **PASS** (Pickup RO-000002 + Delivery RO-000003 as restaurant_orders, server-authoritative pricing); QR/kiosk submit NOT run.
- ECOMMERCE → ORDER → COD: **NOT RUN**
- ECOMMERCE → BANK TRANSFER → PROOF → VERIFICATION: **NOT RUN**
- LOYALTY / PROMOTIONS: **Loyalty PASS**; Promotions NOT run.
- ROLES / PERMISSIONS: **NOT RUN** (owner-only session; no cashier/accountant persona available)
- TENANT ISOLATION: **BLOCKED** (no second account)
- SEED → CLEAR → RESEED: **NOT RUN** (destructive; needs seeder-fix deploy first — to prove fresh-seed payroll)
- FRESH-SEED PAYROLL WITHOUT MANUAL CONFIG: **PENDING** (seeder fix shipped `5e3fb69`; prove at Clear→Reseed after deploy)
- SEED IDEMPOTENCY: **PASS**
- DELETE TENANT COMPLETELY: **NOT EXECUTED**

**PRODUCTION READINESS: CONDITIONAL GO.** All P0-risk financial/inventory workflows verified correct on the live app with balanced double-entry accounting and correct inventory (consume/restore exactly once, UOM conversion). Remaining before full GO: deploy `5e3fb69` and prove fresh-seed payroll at Clear→Reseed; run ecommerce COD + bank-transfer-proof, promotions, QR/kiosk submit, restaurant cancel/transfer/split; enforce-test roles/permissions and tenant isolation with a second account; then the destructive Clear/Delete-tenant phases.

REMAINING PRODUCTION BLOCKERS (actual): tenant-isolation P0 test requires a second account (environment); genuine concurrency/oversell test requires concurrent sessions (environment). No open P0/P1 code defects.

---

# Final Secondary / Security UAT (fresh tenant "UAT QA Shop", shop_id 9ea81419-…)

New disposable tenant created via onboarding + Seed Full UAT Data (all modules "ready", 0 errors). Ran the remaining operational/security flows.

| ID | Scenario | Result | Evidence / Issue | Severity |
|----|----------|--------|------------------|----------|
| S1 | QR table ordering `/r/<token>` | **PASS** | Token resolved **Table UAT-Table-1** (no ID typing); required-modifier sheet (single groups); submitted **RO-000001** (order_type dine_in, channel **qr**, $650); table went "occupied" | — |
| S2 | Kiosk `/r/<token>?kiosk=1` | **PASS** | **Table UAT-Table-2** context, touch layout + persistent panel; submitted **RO-000002**; post-submit reset ("Start a new order") | — |
| S3 | Restaurant cancel + reservation release | **PASS** | Dine-in RO-000003, KOT → reservation Bun 1 / **Patty 0.15** / Sauce 20; **Cancel** → order status **cancelled**, reservations **0**, patty unchanged [3,3], **no consumption movements**, table freed | — |
| S4 | Table transfer | **NOT SUPPORTED** | Server fn `transferTable` exists but **no UI calls it** (only Merge + Split wired in RestaurantPosPanel). Did not fake. | P2 |
| S5 | Restaurant split / partial payment | **NOT SUPPORTED** | Settle dialog is single-method full payment; no payment-allocation/partial UI (the "Split" toolbar splits the bill into separate orders, not payment) | P2 |
| S6 | Ecommerce COD | **PASS** | **ORD-000001** (channel online, **payment unpaid** correct for COD, $120 server-authoritative, customer associated, inventory Main Store 80→79). Only COD + Bank transfer offered — **no card gateway**. | — |
| S7 | Bank transfer payment proof | **FAIL — P1** | Upload **correctly does NOT auto-verify** (order stayed `unpaid`) ✓; proof stored in private `payment-proofs` bucket. BUT merchant **"Verify & mark paid" fails**: `Not a member of this shop`. Root cause: live DB runs the **pre-hardening** `record_order_payment` RPC (guard `if not is_shop_member(_shop_id)` fails for the service-role caller whose `auth.uid()` is null). The corrected RPC already exists in the repo (`20260902100000_harden_record_order_payment.sql`, guard `auth.uid() is not null and not is_shop_member`) but is **not applied to the live database**. **Resolution: apply that migration to live — no code change needed.** | **P1** |
| S8 | Promotions at checkout | **NOT RUN** | Ecommerce checkout exposes no coupon/promo-code field; promotions/coupons seeded but no wired customer-entry point found this pass | — |
| S9 | Recruitment pipeline transition | **PASS** | Moved UAT-Candidate-A **interview → offer** (UAT-Chef-Vacancy); persisted (`job_applications.stage=offer`) | — |
| S10 | Performance transition | **PASS** | UAT-Performance-Cycle **active → review**; persisted (`performance_cycles.status=review`) | — |
| S11 | Loans / advances | **PARTIAL** | UAT-ADV-001 (Bilal, advance, principal **$20,000**, repayment schedule present); draft → **approve** works; **Issue-to-ledger blocked**: "Configure the Advance Receivable account in payroll settings" — seeder doesn't map that account | P2 |
| S12 | Budgets vs Actual | **PASS** | Budget vs Actual from live ledger: **5000 COGS Budget $10,000 / Actual $140.00 / Var −98.6%** — the $140 actual traces to the two ecommerce orders' posted COGS (2 × $70 water bottle); 6000 Op-Ex $20,000/$0 | — |
| S13 | Storage upload / private access | **PASS** | Bank-transfer proof PNG uploaded to the **private `payment-proofs` bucket** (shop-prefixed path); upload succeeded and did **not** auto-verify the order. (Same upload mechanism backs product-images/hr-documents.) | — |
| S14 | Roles / permissions enforcement | **BLOCKED** | Only the owner account is available; cannot create/log-in as cashier/accountant/manager personas to test access denial. Not inferred from constants. | BLOCKED |
| S15 | Tenant isolation (P0) | **BLOCKED** | Only one auth account (owner of all shops); a genuine cross-tenant test needs a second account that is NOT a member of the target shop. Not inferred from RLS. | BLOCKED |
| S16 | Oversell / concurrency (P0) | **BLOCKED** | Genuine simultaneous execution cannot be reliably produced via sequential automation, and exactly-1-portion setup needs an inventory adjustment. **Related evidence:** the ecommerce checkout **correctly rejected overselling** twice ("Insufficient stock: this sale would drive inventory negative") for zero-stock products — real point-of-sale oversell protection, though not a concurrency race. | BLOCKED |

## FINAL VERDICT (secondary/security pass)

**LIVE UAT STATUS: CONDITIONAL GO**

- **P0 OPEN: 0**
- **P1 OPEN: 1** — Bank-transfer proof **verification** fails on live because migration `20260902100000_harden_record_order_payment.sql` is **not applied** to the live DB (the corrected RPC already exists in the repo; **apply the migration** — no code change). The security-critical half (upload ≠ auto-verify) passes.
- **P2 DEFERRED:** (a) Loans "Advance Receivable" account not seeded (blocks Issue out of the box — same class as the payroll-account fix, report-only per instruction); (b) Table transfer has backend `transferTable` but no UI; (c) Restaurant split/partial-payment not in the settle UI; (d) Promotions have no customer coupon-entry at ecommerce checkout.
- **BLOCKED (environment):** Roles/permissions enforcement, Tenant isolation, genuine Oversell/Concurrency — all require a second auth account / persona or true concurrent sessions.

### Secondary core workflows
- QR ORDERING: **PASS**
- KIOSK ORDERING: **PASS**
- RESTAURANT CANCEL: **PASS**
- TABLE TRANSFER: **NOT SUPPORTED (no UI)** — P2
- SPLIT / PARTIAL PAYMENT: **NOT SUPPORTED (no UI)** — P2
- ECOMMERCE COD: **PASS**
- BANK TRANSFER PROOF: **FAIL — P1** (verify blocked; upload-no-auto-verify PASS; fix = apply existing migration)
- PROMOTIONS: **NOT RUN** (no checkout coupon entry found)
- RECRUITMENT: **PASS**
- PERFORMANCE: **PASS**
- LOANS: **PARTIAL** (approve works; issue blocked by unmapped Advance Receivable — P2)
- BUDGETS: **PASS**
- STORAGE: **PASS**
- ROLES / PERMISSIONS: **BLOCKED**
- TENANT ISOLATION: **BLOCKED**
- OVERSALE / CONCURRENCY: **BLOCKED** (oversell-at-sale protection evidenced separately)

**PRODUCTION READINESS: CONDITIONAL GO.** All core financial/inventory workflows and most secondary operational flows pass on the live app. One P1 (bank-transfer verify) is resolved by **applying the already-committed migration `20260902100000`** — then live-retest Verify & mark paid. Remaining P2s are report-only. The three BLOCKED security tests (roles, tenant isolation, concurrency) require a second account/persona or concurrent sessions.

REMAINING PRODUCTION BLOCKERS (actual):
1. Apply migration `20260902100000_harden_record_order_payment.sql` to the live DB, then retest bank-transfer proof verification (P1).
2. Provide a second auth account/persona to run roles-enforcement + tenant-isolation (P0-class security) and true-concurrency oversell.
3. (Optional P2) seed the Advance Receivable payroll account; wire table-transfer/split-payment UI; add a checkout coupon field.

Note: the disposable tenant **UAT QA Shop** was intentionally left in place (not deleted) per the instruction to keep it until all remaining tests are finished.

---

# Destructive lifecycle + seeder-fix proof (Clear → Reseed → Delete) — all live on test2

## PHASE 24 — Clear ALL Tenant Data: **PASS**
Confirmed via typed-name ("test2"). Result **OK**: Database rows deleted **242**, storage files 0, storage failures 0, buckets none, orphans 0, **Residual tenant rows 0**, **Residual tenant files 0**, **Store kept: yes**, **Login kept: yes**. Seed status → "Not seeded". (Note: clearing briefly removed the shop row, so the admin shop-switcher fell back to another shop the owner holds; switching back to test2 showed the restored, empty tenant.)

## PHASE 24 — Fresh Seed Full UAT Data: **PASS**
Reseed created all modules fresh ("N created · 0 reused"). Payroll section message now reads "…**GL account mappings, cash account**, open period & salary structures (base + HRA + Transport)…". Verified fresh config: `cash_accounts` = **UAT-Cash Drawer**; departments 6, products 12 present. (Employees/payroll_settings are server-only RLS, invisible to the anon client by design — proven instead by the payroll post below.)

## FRESH-SEED PAYROLL WITHOUT MANUAL CONFIG: **PASS** ✅ (proves seeder fix #1)
On the freshly-seeded tenant, created run → **Calculate** → **Approve** → **Post to accounting** with **zero visits to Payroll Settings**. Posted OK (status "Posted · 30 Sept 2026"). Journal **PAY-00001**: total debit **$420,000** = total credit **$420,000**; net **6100 Salaries +$420,000 / 2000 Accounts Payable −$420,000**. Balanced.

## BASE-SALARY DEDUP: **PASS** ✅ (proves seeder fix #2)
Fresh calculation shows each employee **Gross $70,000** (base $50,000 + HRA $15,000 + Transport $5,000), total **$420,000** — no duplicate basic-salary component (was $120k/$720k before).

## PHASE 25 — Seed Idempotency (2nd seed on fresh tenant): **PASS**
Re-running Seed = "0 created · N reused" across modules. Post-2-seeds counts: burger `product_modifier_groups` = **exactly 2** (modifier-dup fix holds), departments 6, tables 4, customers 5. No duplicates.

## PHASE 26 — Delete Tenant Completely: **PASS**
Confirmed via typed-name. Result **OK**: Database rows deleted **212**, residual rows 0, residual files 0, **Store kept: no**, **Login kept: yes**. Verified: test2 shop row is **gone** (`shops` where id=6646e8da → `[]`); test2 residual departments **0**; owner's **other shops unaffected** (test, Trendies Store, AVX still present); Supabase Auth account intact.

## FINAL VERDICT
**LIVE UAT STATUS: CONDITIONAL GO** (effectively GO for the tested surface; two environment-gated tests remain).

- P0 OPEN: **0**
- P1 OPEN: **0**
- P2: all authorized/found P2s **fixed** (stock location, payroll account mapping, base-salary dedup, modifier-link idempotency); attendance date-window default remains report-only.
- BLOCKED (environment only): TENANT ISOLATION (needs 2nd tenant/account); genuine OVERSELL/CONCURRENCY (needs concurrent sessions).
- NOT RUN (tenant now deleted): ecommerce COD + bank-transfer proof, promotions at checkout, QR/kiosk submit, restaurant cancel/transfer/split, recruitment/performance/loans/budgets transitions, roles/permissions enforcement, storage upload/retrieval.

### Core workflows (final)
- HR → PAYROLL → ACCOUNTING: **PASS** (incl. fresh-seed post with no manual config; payment no double-post)
- PURCHASING → INVENTORY → AP → ACCOUNTING: **PASS** (GRN→invoice→AP→payment, balanced)
- RETAIL POS → INVENTORY → ACCOUNTING → CRM: **PASS** (+ loyalty earn; + return/refund reversal)
- RESTAURANT → KOT → RESERVATION → SETTLEMENT → INVENTORY → ACCOUNTING: **PASS** (consume-once, UOM g→kg)
- RESTAURANT CUSTOMER ORDERING: **PASS** (Pickup + Delivery as restaurant_orders, server pricing)
- ECOMMERCE → COD / BANK PROOF: **NOT RUN**
- LOYALTY: **PASS** / PROMOTIONS: **NOT RUN**
- ROLES / PERMISSIONS: **NOT RUN** (owner-only session)
- TENANT ISOLATION: **BLOCKED** (env)
- SEED → CLEAR → RESEED: **PASS**
- FRESH-SEED PAYROLL WITHOUT MANUAL CONFIG: **PASS**
- SEED IDEMPOTENCY: **PASS**
- DELETE TENANT COMPLETELY: **PASS**

**PRODUCTION READINESS: CONDITIONAL GO.** All P0-risk financial/inventory workflows verified correct and balanced on the live app; the full Seed→Clear→Reseed→Delete lifecycle works with zero residual and preserves other tenants + the auth account; both seeder P2 fixes proven on a fresh tenant. Only environment-gated tests (multi-tenant isolation, true concurrency) and a set of lower-risk operational flows remain un-executed. No open P0/P1 code defects.

---

# P1 RETEST — Bank-transfer verify after user reported migration applied (2026-09-13)

Context: user manually applied `20260902100000_harden_record_order_payment.sql` to live and asked to re-run the merchant "Verify & mark paid" step on **UAT QA Shop / ORD-000002**.

## Result: **STILL FAILING — P1 REMAINS OPEN** (migration did NOT take effect on live)

Before (authenticated REST, ground truth):
- Order ORD-000002 (id f320c2d5-…): payment_status **unpaid**, status confirmed, paid_total **0**, total $120, shop 9ea81419-… (correct)
- payment_proofs: status **submitted**, verified_by null, verified_at null
- payments: **0 rows**

Action: Admin → Orders → ORD-000002 → Payment Proof → **Verify & mark paid**.

Observed live error (thrown from the server-fn response):

```
Error: Not a member of this shop
```

After (re-read, ground truth) — **no writes occurred** (atomic reject):
- Order: payment_status **unpaid**, paid_total **0** (unchanged)
- payment_proofs: status **submitted**, verified_at null (unchanged)
- payments: **0 rows** (unchanged) — no partial/duplicate posting

## Root cause (conclusive)
The string `Not a member of this shop` exists in the codebase **only** in the OLD migration `20260902074952_cace9501-…sql` (guard `if not public.is_shop_member(_shop_id)`). The corrected function in `20260902100000_harden_record_order_payment.sql` replaces the body and raises a different message (`Not authorized for shop %`) with a service-role-safe guard (`if auth.uid() is not null and not public.is_shop_member(...)`). Because the live function still emits the OLD message, the live `public.record_order_payment` is **still the pre-hardening body** — i.e. migration `20260902100000` has **not actually taken effect on the live database**, despite being reported applied. The app calls the RPC via the service-role `supabaseAdmin` client (auth.uid() null), so the old guard always rejects. App code and the repo migration are both correct — **no code defect.**

Most likely mechanism: version `20260902100000` was already recorded in the live `supabase_migrations.schema_migrations` history (from Lovable's earlier environment), so `supabase db push` reports "up to date" and **skips executing** the SQL, leaving the old function installed.

## Remediation (live-DB only; NO repo change, NO new migration)
Run the idempotent function body **directly in the Supabase SQL editor** for project `tgaibydudmsslfxyzipz` — the exact `create or replace function public.record_order_payment(...)` already committed as `20260902100000` (guard `if auth.uid() is not null and not public.is_shop_member(_shop_id) ... raise 'Not authorized for shop %'`), followed by `grant execute on function public.record_order_payment(uuid,uuid,numeric,text,text,jsonb,uuid) to service_role;`. Running the body directly bypasses the migration-history skip. Then re-run this retest — expected: verify succeeds, proof → verified, order → paid, one `payments` row + one balanced `journal_entries` (Dr 1100 Bank / Cr 1150 AR $120); reload must not double-post.

## Downstream tests blocked by this P1 (not executed this pass)
- **Payment-verify idempotency** (double-click / reload) — cannot test until verify succeeds once.
- **Other `record_order_payment` call sites** — same RPC, same block.

## Second-account / roles / isolation / concurrency
- A staff-**invite workflow exists** (`/admin/team` → `inviteStaff` → `/invite/$token` accept → `shop_users` row with role); personas are a product capability, not a missing feature.
- Completing a **second authenticated session** requires a separate real login (fresh signup + email confirmation, or an invited user's credentials), **not available in this environment**; fabricating auth users / bypassing auth is disallowed. → **Roles/permissions, Tenant isolation, and true Concurrency remain BLOCKED (environment)**, not inferred PASS.

## Destructive lifecycle (Clear → Reseed → Delete) — **NOT EXECUTED this pass (deliberate)**
UAT QA Shop was **kept**. Deleting it now would destroy the only live repro of the open P1; the disposable tenant must remain until the live function is fixed and the bank-transfer verify passes. (The full Clear→Reseed→Delete + seed-idempotency + fresh-seed-payroll lifecycle was already proven PASS on tenant `test2` — see the section above.)

## Updated status
- **LIVE UAT STATUS: CONDITIONAL GO**
- **P0 OPEN: 0**
- **P1 OPEN: 1** — bank-transfer verify (live `record_order_payment` still pre-hardening; fix = run the committed function body directly in the live SQL editor; no code change)
- **P2 DEFERRED (report-only):** Advance-Receivable seed mapping; table-transfer no UI; split/partial-payment no UI; checkout coupon field missing
- **BLOCKED (environment):** Roles/permissions, Tenant isolation, true Concurrency/oversell
- **BANK TRANSFER PROOF: FAIL — P1 (retest)** · upload-no-auto-verify still PASS
- **PRODUCTION READINESS: CONDITIONAL GO** — one live-DB remediation stands between here and closing the last P1.

---

# P1 RESOLVED — Bank-transfer verify PASS after live RPC repair (2026-09-14)

User repaired the live `public.record_order_payment` directly in the Supabase SQL editor (verification `is_hardened = true`). Re-ran the full merchant verify + accounting + idempotency on **UAT QA Shop / ORD-000002**.

## 1. Verify action — **PASS**
Admin → Orders → ORD-000002 → Payment Proof → **Verify & mark paid**: succeeded with **no error** (previously threw `Not a member of this shop`). UI updated to Payment **paid**, proof **verified**, and exposed **Return / refund items** (payment now exists).

## 2. Live DB after verify — **PASS**
| Record | Before | After |
|--------|--------|-------|
| orders.payment_status | unpaid | **paid** |
| orders.paid_total | 0 | **120** (total 120) |
| payment_proofs.status | submitted | **verified** |
| payment_proofs.verified_at / verified_by | null / null | **2026-09-13T21:33:42Z / f4cc491f-… (owner)** |
| payments (count) | 0 | **1** |

Payment row: id 992cc5f9-…, amount **120**, method_code **bank**, reference "Account transfer", status **captured**, created_by f4cc491f-… (owner).

## 3. Accounting — **PASS** (balanced, correct accounts)
Payment journal `6039c59d-…`, memo **"Order payment"**, reference_type=order, reference_id=ORD-000002:
- Dr **1100 Bank** 120
- Cr **1150 Accounts Receivable** 120
- SUM(debit)=SUM(credit)=**120** ✓ — exactly the expected Bank Dr / AR Cr settlement.

(The earlier order-placement entry `1cd89c3a-…` memo "Sale" — Dr 1150 AR 120 / Cr 4000 Sales 120 and Dr 5000 COGS 70 / Cr 1200 Inventory 70 — is unchanged and balanced. The verify correctly clears the AR raised at placement.)

## 4. Idempotency — **PASS** (no double-post)
Server guard [reviewPaymentProof](src/lib/payment-proofs.functions.ts) only calls `applyOrderPayment` when `payment_status !== "paid" && remaining > 0.001`; once paid, a re-verify re-stamps the proof but posts nothing. UI corroborates: after **page reload**, the "Verify & mark paid" button is **absent** (only Preview + Return/refund). Post-reload counts unchanged: **payments = 1**, **"Order payment" journals = 1**, order still paid/120, proof still verified & traceable (Preview works).

## Updated status
- **LIVE UAT STATUS: GO** (for the tested surface)
- **P0 OPEN: 0** · **P1 OPEN: 0** ✅ (bank-transfer verify now **FIXED BY LIVE MIGRATION + LIVE RETEST PASS**)
- **BANK TRANSFER PROOF: PASS** — upload-no-auto-verify PASS + merchant verify PASS + balanced accounting + idempotent
- **P2 DEFERRED (report-only, unchanged):** Advance-Receivable seed mapping; table-transfer no UI; split/partial-payment no UI; checkout coupon field missing
- **BLOCKED (environment, unchanged):** Roles/permissions, Tenant isolation, true Concurrency/oversell (need a second real auth session)
- No application code changed (the fix was a live-DB function repair the user applied); tsc/Vitest/build baseline from `5e3fb69` stands.
- UAT QA Shop **kept** (not deleted), per instruction.

---

# Cross-tenant isolation — LIVE with a genuine second account (2026-09-14)

Second authenticated account provided by the user (password never recorded). Logged in via the real Supabase Auth password grant + UI.

**Second account identity (authoritative):** user `40a529af-5778-48c9-9ce2-401ccd1ba643`, email confirmed, and — per `shop_users` read with *its own* token — a member (owner) of **exactly one** shop `5c9a4c26-086b-4bdb-8ea2-981fe1330c87` ("Trendies Store (Haroon Shopping Center)"). It is **NOT** a member of UAT QA Shop `9ea81419-…`. This makes it a valid non-member for isolation testing.

## 1. Cross-tenant READ (second account's real JWT → UAT QA Shop) — **PASS**
Direct PostgREST reads (bypasses UI nav — this is direct-data access, exactly what the instruction requires) of UAT QA Shop rows returned **0 rows** for every protected table:
`orders 0, payments 0, payment_proofs 0, customers 0, employees 0, journal_entries 0, journal_entry_lines 0, restaurant_orders 0, restaurant_tables 0, departments 0, cash_accounts 0, accounts 0, suppliers 0`.
Only `products`/`product_variants` were readable — but **identical to an anonymous (no-login) request** (anon key alone returns the same 12 active products), i.e. the **intended public storefront catalog**. Exposed columns are catalog/selling fields only (name, slug, status, price, compare_at_price); **no `cost`/COGS, no supplier linkage** (`suppliers` = 0). → Not an isolation defect; no sensitive data crosses tenants.

## 2. Cross-tenant WRITE (second account's real JWT → UAT QA Shop) — **PASS**
- UPDATE UAT QA order (set status=cancelled): **403** `42501 insufficient_privilege` — denied.
- DELETE UAT QA order: **403** — denied.
- INSERT a new order into UAT QA Shop: **403** — denied.
- UPDATE a UAT QA product name (public-read table): **0 rows affected** — RLS allows public SELECT but blocks the write; product unchanged.

## 3. Cross-tenant STORAGE (private `payment-proofs`) — **PASS**
UAT QA Shop's proof object `9ea81419-…/f320c2d5-…/….png`:
- Non-member direct download: **400** denied.
- Non-member create-signed-URL: **404 not_found** (object hidden by RLS from the non-member).
- Anonymous download: **400** denied.

## 4. UI-level isolation — **PASS**
Logged into the UI as the second account: its **"Your shops"** switcher lists **only** "Trendies Store (Haroon Shopping Center)" (owner). **UAT QA Shop is absent** and cannot be selected, so there is no in-app path to another tenant's admin context. (The authoritative direct-data denial is proven in §1–§3 above, not merely by hidden navigation.)

## Verdict
**TENANT ISOLATION: PASS** (was BLOCKED). No cross-tenant read/write/storage access is possible for a genuine non-member; the only cross-tenant-visible data is the deliberately public storefront product catalog (no cost/supplier/financial fields). **No isolation defect found — severity N/A.**

## Roles/permissions, staff-invite role scenarios, concurrency — **BLOCKED THIS PASS (owner session lost mid-run)**
A transient host `net::ERR_NETWORK_CHANGED` during the isolation run caused the **owner (UAT QA Shop) client session to be signed out** (localStorage auth token cleared; page redirected to `/auth`). The owner password is not held by the tester, so the owner session could not be restored.
These remaining tests **require the UAT-QA-Shop owner session** and therefore could not be completed:
- **Staff invite + role enforcement** — needs the owner to invite the second account into UAT QA Shop with a limited role (cashier/accountant) so denials can be exercised as that role.
- **Concurrency / last-portion oversell** — needs an owner/admin action to set a product's sellable stock to exactly 1 before two independent sessions race for it.
Action needed to unblock: the UAT-QA-Shop owner logs back in (or provides credentials) — then role-enforcement (allowed vs denied actions via real login + direct route/server-action) and the two-session concurrency test can be executed. Not inferred PASS.

### Follow-up (2026-09-14): a genuine second account was provided; owner setup still could not be arranged
A real second account (`sarimulhaq74`) was supplied and used to complete **cross-tenant isolation = PASS** (above). For role-enforcement + concurrency, the UAT-QA-Shop **owner** session was still required (to invite the second account into UAT QA Shop with a restricted role, and to set a product's sellable stock to exactly 1). This could not be arranged in the environment:
- The two integrated-browser tabs **share one session** (localStorage), so they cannot hold two different users at once.
- The owner session had been signed out (network blip) and the owner credentials were not available to the tester.
- The owner's separate browser page could not be shared with the agent.
Per the user's decision, **ROLES/PERMISSIONS and CONCURRENCY/OVERSELL are recorded as BLOCKED (environment)** for this engagement. They are not defects and were not inferred PASS. To close them later: (a) owner invites `sarimulhaq74` to UAT QA Shop as Cashier/Accountant → log in as that role → attempt payroll/HR/danger-zone via real login **and** direct route/server-action (all must be denied); (b) owner sets one product's sellable stock to 1 → fire two concurrent `placeOnlineOrder` requests → expect exactly one success, `quantity - reserved >= 0`, no double-sell.

## FINAL CONSOLIDATED STATUS (2026-09-14)
- **P0 OPEN: 0** · **P1 OPEN: 0**
- **BANK TRANSFER PROOF: PASS** (live retest — verify succeeds, order paid, 1 payment, balanced Bank Dr / AR Cr journal, idempotent on reload)
- **TENANT ISOLATION: PASS** (genuine non-member second account — reads/writes/storage all denied; only public storefront catalog visible, no cost/supplier fields)
- **ROLES / PERMISSIONS: BLOCKED (environment)** — needs owner to seat a restricted-role member
- **CONCURRENCY / LAST-PORTION OVERSELL: NOT IN CURRENT UAT SCOPE — intentionally skipped by user decision** (not a blocker, not a release defect; non-concurrent oversell-at-sale protection is separately evidenced)
- **P2 (report-only, unchanged):** Advance-Receivable seed mapping; table-transfer no UI; split/partial-payment no UI; checkout coupon field missing
- **PRODUCTION READINESS: GO for the fully-tested surface**, with two environment-gated security tests (role enforcement, true concurrency) still to be executed once an owner-seated restricted role and a controlled stock=1 product are available. No open P0/P1 code defects.
- UAT QA Shop **kept** (not deleted).

---

# Targeted live migration-application audit (2026-09-14)

Prompted by the `record_order_payment` incident (migration marked-applied but stale on live). Audited the effective **live** DB (project `tgaibydudmsslfxyzipz`) against the authoritative migration sources, via REST schema/RPC probes (schema-level errors surface regardless of RLS) + behavioral evidence from this cycle. **No blind re-runs.**

## Tables — 18/18 VERIFIED LIVE
`payment_proofs`, `restaurant_floors`, `restaurant_tables`, `restaurant_table_sessions`, `restaurant_menu_items`, `restaurant_orders`, `restaurant_order_items`, `restaurant_order_item_modifiers`, `restaurant_settings`, `restaurant_order_item_voids`, `restaurant_bill_splits`, `restaurant_bill_split_items`, `restaurant_bill_split_payments`, `restaurant_session_tables`, `restaurant_reservations`, `restaurant_waitlist`, `restaurant_external_orders`, `restaurant_ingredient_reservations` → all **EXISTS**.

## Additive columns — 22/22 VERIFIED LIVE
- `restaurant_orders`: delivery_address, delivery_phone, delivery_fee, delivery_status, delivery_notes → OK (migration `20260914100000_restaurant_complete`)
- `restaurant_settings`: default_delivery_fee, **menu_availability_mode**, **low_stock_threshold** (`20260918100000`), **public_ordering_enabled**, **pickup_enabled**, **delivery_enabled**, **public_ordering_token** (`20260919100000`) → OK
- `payment_proofs`: status, verified_by, verified_at, file_path, content_type → OK
- `restaurant_ingredient_reservations`: restaurant_order_id, location_id, variant_id, quantity (`20260920100000`) → OK

## RPCs — 9/9 critical VERIFIED LIVE
Probed with correct argument signatures; service-role-only functions correctly return `403 permission denied` to the authenticated role (proves existence + correct lockdown):
- `has_shop_role`, `is_shop_member` → EXISTS (200)
- `record_order_payment` → EXISTS **+ hardened** (today's bank-transfer verify PASS is behavioral proof the new guard body is live; `is_hardened=true` per user's catalog check)
- `record_pos_sale` → EXISTS (403) + behaviorally verified live (ORD-000001 COD write)
- `record_pos_return` → EXISTS (403)
- `apply_inventory_delta` → EXISTS (403)
- `apply_restaurant_reservation`, `release_restaurant_reservation` → EXISTS (403) + behaviorally verified live (RO-000003 reserve on KOT, release on cancel)
- `next_document_number` → EXISTS (403) + behaviorally verified live (all order/RO/receipt numbers)

## Classification
- **MATCH / VERIFIED LIVE:** all 18 tables, all 22 additive columns, all 9 critical RPCs above.
- **STALE / REPAIR REQUIRED:** none found (beyond the already-repaired `record_order_payment`, which is now confirmed live-correct).
- **MISSING:** `increment_coupon_usage` — **not defined in ANY migration** (only *called* in `marketing.server.ts`, and that call swallows errors via `.then(r=>r, ()=>null)`). This is **not** a stale-apply regression; it belongs to the **unwired promotions/coupons** feature (already a report-only P2, out of current scope). Non-breaking (checkout unaffected). **No repair applied** (creating it would be adding an out-of-scope feature).
- **NOT APPLICABLE:** none.

**Audit verdict:** the live schema/functions the application depends on are present and current. No P0/P1 migration-staleness defects remain. The single missing RPC is a known unwired-promotions gap, not a regression, and is intentionally left report-only.

---

# Seeder fix, type-shim review & baseline (2026-09-14)

## Advance/Loan Receivable seeder mapping — CODE FIXED (live validation pending deploy)
[uat-seed.server.ts](src/lib/uat-seed.server.ts) `seedPayroll` now maps the HR receivable + funding accounts on `payroll_settings`, resolved by authoritative COA **code** (no hardcoded IDs), **backfill-only** (never clobbers a real merchant's mapping), idempotent:
- `loan_receivable_account_id` → `1150` Accounts Receivable
- `advance_receivable_account_id` → `1150` Accounts Receivable
- `loan_funding_account_id` → `1000` Cash
This makes employee advances/loans **Issue & Post** work out of the box (issue posts Dr receivable / Cr funding). `tsc` clean; `uat-seed-spec` tests pass (20/20). Committed `4a79073`.
- **Live validation status:** attempted on a fresh disposable tenant ("UAT Loan QA", `39bd7e73-…`, owner account). Two blockers prevented a live PASS this session: (1) **deploy of `4a79073` could not be confirmed** — the change is server-only (no client-bundle change to fingerprint) and the seed result message is a transient toast; (2) **host network instability** — repeated `ERR_INTERNET_DISCONNECTED`/`ERR_NETWORK_CHANGED` on the browser host caused React Query to retry/refetch in a loop, so the Loans table never became "stable" and the loan-detail modal / Approve / Issue mutations would not persist (loan stayed `draft`; direct DB reads confirmed no status change and no journal). The `payroll_settings` mapping itself can't be read via REST (server-only RLS returns `[]`). **Conclusion: the fix is committed + unit-validated (tsc + 20/20 seed tests); the live loan Issue&Post retest is BLOCKED by the environment (unconfirmed deploy + host network), not by code.** To complete: confirm `4a79073` is deployed, then on a stable connection do loan **Approve → Issue & post** and verify `employee_loans.status=active`, `issued_journal_entry_id` set, and a balanced journal **Dr 1150 Accounts Receivable / Cr 1000 Cash $20,000**, with no double-post on reload. On success, promote this P2 to **FIXED + LIVE UAT PASS**.
- Note: one extra disposable shop, **"UAT Loan QA"**, was created for this validation and remains; it should be deleted (Danger Zone → Delete Tenant Completely) once validation is done or abandoned.

### Live retry (2026-09-14, after user confirmed deploy + stable network)
Retried end-to-end on **UAT Loan QA** (`39bd7e73-…`). Findings:
- **Approve works** — loan `draft → approved` persisted (UI + DB row updated). So network/mutations are fine.
- **Issue & post STILL FAILS** with `Configure the Advance Receivable account in payroll settings before issuing.` — repeated **after multiple re-seeds + a propagation wait**.
- **Precondition verified present:** account **`1150` Accounts Receivable exists** for the shop (owner REST read), so the seeder's `accountByCode("1150")` would resolve it if the new code ran.
- **No manual workaround exists:** the Payroll → Settings tab exposes only salary-expense / employee-payable / deduction accounts; there is **no UI field** for Advance/Loan Receivable or funding (the "Receivables" tab is a customer-AR ageing report). So the mapping can ONLY be set by the seeder (my fix) or a direct DB write — **which elevates this from "convenience" to "required": without the seeder fix (or a new settings field), a fresh tenant cannot issue advances/loans at all.**
- **Conclusion (definitive):** the re-seed does **not** set `advance_receivable_account_id` even though `1150` exists → the **live server is still running the pre-`4a79073` seeder**. The deploy of `4a79073` has **not propagated to the running server**, despite being reported deployed (same deploy-lag class as the earlier RPC repair, which also needed direct action). The fix is correct, committed, and unit-validated; the ONLY remaining step is an effective deploy. Once the server actually runs `4a79073`: re-seed → the mapping is set → Issue posts a balanced **Dr 1150 / Cr 1000 $20,000** journal. **UAT Loan QA is intentionally kept** so this can be re-validated the moment the deploy lands.

### ✅ RESOLVED — LIVE UAT PASS (2026-09-14, after full worker redeploy)
After the user performed a full server/worker redeploy, re-validated on **UAT Loan QA**. **Deploy now confirmed live** — the seed's payroll result message reads: *"GL account mappings **(incl. HR loan/advance receivable + funding)**…"* (the new wording from `4a79073`). (Root-cause of the earlier "still failing" retries: the re-seed clicks were being auto-cancelled by the confirm dialog, so the new seeder never actually ran — once the seed genuinely executed, the backfill set the mapping.)
End-to-end result on **UAT-ADV-001** (advance, principal $20,000):
- Create (seeded) → **Approve** (`draft → approved`) → **Issue & post** → **SUCCESS, no error**.
- `employee_loans`: status **active**, outstanding **$20,000**, `issued_journal_entry_id` set.
- Journal **JE-00001** (memo "Advance UAT-ADV-001", reference_type `employee_loan`): **Dr `1150` Accounts Receivable $20,000 / Cr `1000` Cash $20,000** — balanced (ΣDr = ΣCr = $20,000). ✔ correct Advance Receivable debited, correct cash funding credited.
- Repayment schedule generated: **4 × $5,000 = $20,000**, all pending.
- **Idempotency:** after reload, exactly **1** journal entry for the loan (no double-post); UI no longer offers "Issue" (status active). ✔
- **P2 status: FIXED + LIVE UAT PASS.** Note: there is still no Payroll-Settings UI field for these receivable/funding accounts, so the seeder is the authoritative bootstrap for UAT tenants; a real merchant tenant would need the accounts set via seed or a future settings field (reported, not in scope).

## Type-shim cleanup (`rdb`/`cdb`/`hdb`/`fdb`/`edb`/`rsupa`) — REVIEWED, INTENTIONALLY KEPT
Empirically tested removal by pointing a shim alias at the typed client (`const edb = supabaseAdmin`) in [ess.server.ts](src/lib/ess.server.ts) and running `tsc`. Result: **hard failures**, because the regenerated `types.ts` is **still incomplete**. Confirmed **missing** tables (used by these modules) include: `attendance_records`, `leave_balances`, `attendance_settings`, `product_modifiers`, `recipe_ingredients`. The typed client additionally raised `TS2589: Type instantiation is excessively deep` on the missing-table query paths, and the now-typed return shapes broke the TanStack server-fn `ValidateSerializable` constraint in the paired `*.functions.ts`.
- Per the instruction's guardrail ("if current generated types still do not represent a schema area correctly, keep that shim and document why"), **all category-B shims are retained**. Removing them requires a **complete** Supabase type regeneration that includes the missing tables above — the earlier regeneration was partial. **No shims removed this pass; reverted the experiment.** Recommendation: regenerate `types.ts` from the fully-migrated live schema (it *is* migrated — see the audit above), then re-attempt shim removal.

## Automated baseline — ALL GREEN
- `tsc --noEmit`: **0 errors**
- Vitest: **261 passed / 20 files**
- `vite build`: **success** (nitro/cloudflare-worker `.output/` generated)

---

# P1 RETEST - Bank-transfer verify after user reported migration applied (2026-09-13)

Context: user manually applied `20260902100000_harden_record_order_payment.sql` to live and asked to re-run the merchant "Verify & mark paid" step on **UAT QA Shop / ORD-000002**.

## Result: **STILL FAILING � P1 REMAINS OPEN** (migration did NOT take effect on live)

Before (authenticated REST, ground truth):
- Order ORD-000002 (id f320c2d5-�): payment_status **unpaid**, status confirmed, paid_total **0**, total $120, shop 9ea81419-� (correct)
- payment_proofs: status **submitted**, verified_by null, verified_at null
- payments: **0 rows**

Action: Admin ? Orders ? ORD-000002 ? Payment Proof ? **Verify & mark paid**.

Observed live error (page error thrown from the server-fn response):
```
Error: Not a member of this shop
```

After (re-read, ground truth) � **no writes occurred** (atomic reject):
- Order: payment_status **unpaid**, paid_total **0** (unchanged)
- payment_proofs: status **submitted**, verified_at null (unchanged)
- payments: **0 rows** (unchanged) � no partial/duplicate posting

## Root-cause (conclusive)
The string `Not a member of this shop` exists in the codebase **only** in the OLD migration `20260902074952_cace9501-�sql` (guard `if not public.is_shop_member(_shop_id)`). The corrected function in `20260902100000_harden_record_order_payment.sql` replaces the body and raises a different message (`Not authorized for shop %`) with a service-role-safe guard (`if auth.uid() is not null and not public.is_shop_member(...)`). Because the live function still emits the OLD message, the live `public.record_order_payment` is **still the pre-hardening body** � i.e. migration `20260902100000` has **not actually taken effect on the live database**, despite being reported as applied.

Most likely cause: the version `20260902100000` was already recorded in the live `supabase_migrations.schema_migrations` history (from Lovable's earlier environment), so a `supabase db push` reports "up to date" and **skips executing** the SQL � leaving the old function in place. (The app calls the RPC via the service-role `supabaseAdmin` client; the app code and the repo migration are both correct � no code defect.)

## Remediation (live-DB only; NO repo change, NO new migration)
Run the idempotent function body **directly in the Supabase SQL editor** for project `tgaibydudmsslfxyzipz` (the exact body already committed as `20260902100000`):
- `create or replace function public.record_order_payment(...) ... if auth.uid() is not null and not public.is_shop_member(_shop_id) then raise exception 'Not authorized for shop %' ...`
- then `grant execute on function public.record_order_payment(uuid,uuid,numeric,text,text,jsonb,uuid) to service_role;`

Running the body directly bypasses the migration-history skip. After that, re-run this retest � expected: verify succeeds, proof ? verified, order ? paid, one `payments` row + one balanced `journal_entries` (Dr 1100 Bank / Cr 1150 AR $120), and a reload does not double-post.

## Downstream tests blocked by this P1 (not executed this pass)
- **Payment-verify idempotency** (double-click / reload) � cannot test until verify succeeds once.
- **Manual/other `record_order_payment` call sites** � same RPC, same block.

## Second-account / roles / isolation / concurrency
- A staff-**invite workflow exists** (`/admin/team` ? `inviteStaff` ? `/invite/$token` accept ? `shop_users` row with role). So personas are a *product* capability, not missing.
- However completing a **second authenticated session** requires a separate real login (fresh signup + email confirmation, or an invited user's credentials) which is **not available in this environment**, and fabricating auth users / bypassing auth is explicitly disallowed. ? **Roles/permissions, Tenant isolation, and true Concurrency remain BLOCKED (environment)**, not inferred PASS.

## Destructive lifecycle (Clear ? Reseed ? Delete) � **NOT EXECUTED this pass (deliberate)**
UAT QA Shop was **kept**. Deleting it now would destroy the only live repro of the open P1; the disposable tenant must remain until the live function is fixed and the bank-transfer verify passes. (The full Clear?Reseed?Delete + seed-idempotency + fresh-seed-payroll lifecycle was already proven PASS on tenant `test2` earlier � see the "Destructive lifecycle" section above.)

## Updated status
- **LIVE UAT STATUS: CONDITIONAL GO**
- **P0 OPEN: 0**
- **P1 OPEN: 1** � bank-transfer verify (live `record_order_payment` still pre-hardening; fix = run the committed function body directly in the live SQL editor; no code change)
- **P2 DEFERRED (report-only):** Advance-Receivable seed mapping; table-transfer no UI; split/partial-payment no UI; checkout coupon field missing
- **BLOCKED (environment):** Roles/permissions, Tenant isolation, true Concurrency/oversell
- **BANK TRANSFER PROOF: FAIL � P1 (retest)** � upload-no-auto-verify still PASS
- **PRODUCTION READINESS: CONDITIONAL GO** � one live-DB remediation stands between here and closing the last P1.

---

# FINAL â€” Disposable tenant lifecycle + overall status (2026-09-14)

## Seed idempotency â€” PASS
UAT Loan QA was seeded ~5Ã— (idempotently). Post-multi-seed row counts are the canonical singletons â€” no duplication: departments **6**, products **12**, customers **5**, restaurant_tables **4**, suppliers **2**, employee_loans **1**. (`employees` reads as 0 via REST by server-only RLS design, not missing data â€” the seeded advance references employee "Bilal UAT-Raza".)

## Delete Tenant Completely â€” PASS (both disposable tenants)
- **UAT Loan QA** (`39bd7e73-â€¦`): Delete result **OK** â€” DB rows deleted **238**, storage files **0**, failures **0**, orphans **0**, **residual rows 0**, **residual files 0**, Store kept **no**, Login kept **yes**.
- **UAT QA Shop** (`9ea81419-â€¦`): Delete result **OK** â€” DB rows deleted **236**, **storage files deleted 1** (the payment-proof PNG in `payment-proofs`), failures **0**, orphans **0**, **residual rows 0**, **residual files 0**, Store kept **no**, Login kept **yes**.

**Post-delete verification (authoritative REST):** `shops` rows for both = **0** (gone); residual UAT QA `departments` = **0** (no orphans); **Auth account preserved** (`sameer.ulhaq79@gmail.com` still logged in); **unrelated tenants untouched** â€” owner's remaining shops = **test, Trendies Store**. This is a P0-class destructive-isolation confirmation: deleting a tenant removes only that tenant's data + files, keeps the auth account, and does not affect other tenants.

## OVERALL FINAL STATUS
- **P0 OPEN: 0 Â· P1 OPEN: 0**
- **BANK TRANSFER PROOF: PASS** (verify posts balanced Bank Dr / AR Cr, idempotent)
- **TENANT ISOLATION: PASS** (genuine non-member: all reads/writes/storage denied; only public catalog visible)
- **MIGRATION AUDIT: PASS** (18 tables / 22 columns / 9 RPCs verified live; only unwired `increment_coupon_usage` missing â€” report-only)
- **ADVANCE/LOAN RECEIVABLE SEEDER: FIXED + LIVE UAT PASS** (loan Issue&Post â†’ active, Dr 1150 / Cr 1000 $20,000, idempotent)
- **TYPE-SHIM CLEANUP: reviewed, intentionally kept** â€” regenerated `types.ts` still omits `attendance_records`, `leave_balances`, `attendance_settings`, `product_modifiers`, `recipe_ingredients` (+ typed client hits TS2589); needs a *complete* type regen first.
- **BASELINE: tsc 0 Â· Vitest 261/261 Â· vite build success**
- **SEED IDEMPOTENCY: PASS Â· DELETE TENANT: PASS (0 residual, auth preserved, other tenants safe)**
- **ROLES / PERMISSIONS: BLOCKED** (would need a restricted-role member seated by owner; environment)
- **CONCURRENCY / LAST-PORTION OVERSELL: NOT IN CURRENT UAT SCOPE** (intentionally skipped by user decision â€” not a blocker, not a defect; non-concurrent oversell-at-sale protection separately evidenced)
- **PRODUCTION READINESS: GO** for the tested surface. Only remaining item is role-enforcement, which needs a second seated restricted-role account (environment), not a code defect.

Remaining follow-ups (non-blocking): (1) complete Supabase type regeneration â†’ then remove the category-B shims; (2) optionally add a Payroll-Settings UI field for loan/advance receivable + funding accounts (today only the seeder sets them); (3) role-enforcement live test when a restricted-role account is available; (4) P2 UI items (table-transfer, split/partial payment, checkout coupon field) remain report-only.

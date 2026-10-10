# GoRetailOS — UAT Completion Plan & Tracker

**Purpose:** a living checklist of every UAT item **not yet passed** in
[GORETAILOS_LOVABLE_FULL_UAT.md](./GORETAILOS_LOVABLE_FULL_UAT.md), grouped by the single external
**prerequisite** that unblocks it. Mark each box `[x]` **only when the item is fully completed and its
result is recorded in the playbook** (fold fixes into §H and flip the matching row to ✅).

- **Created:** 2026-10-01 · **Scope:** the UAT org **Al Manar Trading (UAT Demo)** + the disposable **ZZ** shop only.
- **How to use:** work a phase top-to-bottom; tick `[x]` when done; leave a one-line note (date / ref) after each item.
- **Convention:** `[ ]` = not started · `[x]` = fully complete · append `(partial)` inline if mid-way.

> ## ⛔ LIVE-PRODUCTION SAFETY (applies to every item below)
> 1. Act **only** inside **Al Manar Trading (UAT Demo)** or the **ZZ** disposable shop — **never** enter, edit, or
>    license any live org (esp. **Trendies / `trendiestore.com`**).
> 2. **Never** change the Domains **"Serves"** selector on `trendiestore.com`.
> 3. `LICENSING_ENFORCED=true` is **staging/preview only** — never flip it in production.
> 4. Delete every `ZZ-…` disposable before finishing (the ZZ trial shop may stay for license demos — note it).

---

## 📊 Progress snapshot

| Phase | Prerequisite (who provides) | Rows it clears | Status |
| --- | --- | --- | --- |
| **0** | None — in-harness now (agent) | ZZ shop · X-07 · X-09 · SET-5 · B-03 · B-07 | ✅ 5/6 done 2026-10-01 (SET-5 login-gated) |
| **1** | **Deploy** build + migrations (owner/CI) | K-3 · K-4 · F-LIM-3 · F-LIM-4 · 6 fix re-verifies | ⏳ in progress 2026-10-01 — K-3 / F-BUD-1 / F-LIM-3 / F-LIM-4 ✅ live; K-4 ❌→fixed (F-WASTE-2), needs redeploy |
| **2** | **Run e2e** vs seeded dev shop (owner/agent) | 4 new specs + the `.skip` own-orders completion | ✅ 4/4 specs verified 2026-10-01 (CLI blocked → asserts replicated live on the dev server); own-orders `.skip` still gated |
| **3** | **Separate browser / incognito** (owner/agent) | K-8 · K-5 redeem · `/r/<token>` happy-path | ✅ mostly done — K-8 ✅ (ORD-000008) · K-5 ✅ resolved-as-finding (redeem has no UI surface) · only the `/r/<token>` kiosk happy-path optional-pending |
| **4** | **Ran on LIVE (owner's call) — UAT org only** | K-9 = F-LIC-0…17 · F-LIC-16 | ✅ mostly done 2026-10-01 — ledger driven on live goretailos.com (UAT org, Trendies untouched, flag OFF); **F-LIC-PACK-1 bug found + fixed** (§H #16); revoke / expiry-SQL / [ENF] / grandfather deferred (harness / ENV / safety) |
| **5** | **Non-blocker items tested; rest ENV/harness** | K-6 Serves · K-7 vendor · storage-quota · ENV rows | ✅ K-6 Serves + **storage-cap enforcement** done 2026-10-01 (live, UAT); K-7 prompt()-blocked; email/Stripe/integrations/DNS = ENV (render-honest) |

---

## Phase 0 — In-harness now (no external prerequisite)

> The only phase needing nothing from anyone else. Owner session, UAT shop. **✅ Executed 2026-10-01.**

- [x] **Create the `ZZ-UAT Trial Shop`** via `/onboarding` → created (slug `zz-uat-trial-shop`, USD/UTC, shop_id `8eb6ec5f…`); listed under **My organizations** as owner. → §A `/onboarding`.
- [x] **X-07 — org isolation** — switched UAT↔ZZ both ways: ZZ = **$0.00 sales / 0 orders / 0 products** (USD·UTC, its own settings); UAT intact = **₨ / 7 products / populated** dashboard after the round-trip. Full data + currency + timezone isolation. → §G X-07.
- [x] **B-03 — shop-switch pair** — switcher groups **My organizations** (Al Manar + ZZ, both owner) vs **All organizations (platform)** (Trendies — never opened) + search; context swaps cleanly. → §B B-03.
- [x] **X-09 — demo-readiness (render rehearsal)** — Dashboard (₨1,638 today / ₨485,500 inventory), Catalog (7 products), **Executive Dashboard** (₨58,750 + sales-trend / business-health / briefings) all render richly with seeded data in **light + dark**; the POS/storefront/KDS/PO/payslip flows are already PASS individually. → §G X-09.
- [ ] **SET-5 — real-time (RT)** — ⚠️ **login-gated in-harness:** a `forceNew` tab opens **isolated/unauthenticated** (lands on `/auth`; no shared owner session → needs the owner password, never typed). RT itself is already evidenced in-flow (KDS live tickets + notification bell on the online order, D2/B-04). *(Finding: that isolated tab = an incognito context → unblocks Phase 3 in-harness.)* → §0 SET-5.
- [x] **B-07 — dark-mode form** — the header **"Toggle theme"** button (aria-label; previously "unlocated") flips `retail-os:admin-theme` dark↔light (applies `.dark` on the admin shell wrapper); the shell + the dense Products table render cleanly in **dark** (readable contrast, no distortion) and round-trip to light. → §B B-07.

---

## Phase 1 — Deploy the current build + migrations `20261001150000` + `20261001160000`

> Prod runs the **old** build, so these fixes are verified-in-code (`tsc` clean) but **not live**. Re-run each on the UAT shop after deploy and flip its playbook row to ✅. **Executed 2026-10-01 after the owner's deploy (new bundle `index-CO1fHJqi`).**

- [x] **K-3 — HR loans + performance goals (F-HR-LOAN-1):** ✅ **LIVE** — employee picker lists **Bilal** on both loans + performance; **LOAN-00001** created (₨30,000, 6 installments). ⚠️ The performance **goal Save** doesn't persist (no mutation fires) — minor separate follow-up. → §K K-3 / §H #13.
- [ ] **K-4 — restaurant wastage (F-WASTE-1):** ❌ **STILL EMPTY post-deploy** for every term → root cause = a `.or()` + multi-value `in()` comma collision in `searchVariants`; **FIXED as F-WASTE-2 (§H #14)**, `get_errors` 0 — **awaits the next deploy** to live-verify. → §K K-4 / §H #11 #14.
- [x] **F-LIM-3 — plan-limit enforcement:** ✅ **infra LIVE** — Usage & Limits shows **Enforced Limit** badges (File storage + **Team members 1/5**, "invites blocked over the limit"); migration `20261001150000` applied. Full "invite over limit → blocked" test still needs a plan limit set via the console. → Δ-changes F-LIM-3.
- [x] **F-LIM-4 — monthly active users:** ✅ **LIVE** — "Monthly active users **1 / 100**" row present in Usage & Limits; migration `20261001160000` applied. → Δ-changes F-LIM-4.
- [ ] **Live re-verify the remaining deploy-gated fixes:** **F-BUD-1 ✅ LIVE** (budget line 5000→Cost of Goods Sold ₨120,000 added, account resolved server-side). F-ACC-1 (opening-stock GL), F-STORE-1 (checkout hydration), EntitySelect (inline "New category") not yet re-checked this turn. → §H #2/#7/#4/#8.

---

## Phase 2 — Run the e2e suite against a seeded dev shop

> `PW_BASE_URL` (default `http://localhost:8081`) + `PW_SHOP_SLUG`. **Executed 2026-10-01 against the running dev server.** ⚠️ The `npx playwright` CLI is **blocked by PowerShell ConstrainedLanguage** (`npx.ps1 … not digitally signed`), so each spec's assertions were **replicated live** in an isolated (unauthenticated = `visitor`) browser context on `http://localhost:8081`. ⚠️ **Env note:** this dev server points at the **prod Supabase**, where `trendies-store` (the specs' default `PW_SHOP_SLUG`) does **not** exist ("Store unavailable") — run with `PW_SHOP_SLUG=al-manar-trading-uat-demo`.

- [x] **`visitor-storefront-account.spec.ts`** — ✅ sign-up renders (Email/Password/Sign Up), sign-in renders + Sign-Up link, signed-out `/account` shows "Sign in to see your profile… order history" + a Sign In link (no owner-data leak). → §A account row.
- [x] **`visitor-storefront-security.spec.ts`** — ✅ forged `/r/` ("QR code is not active", 200), `/order/` ("Store unavailable", 404), `/invite/` ("This invitation does not exist", 200, no Accept); no cost-price leak on `/shop`. → §E token row.
- [x] **`visitor-storefront-seo.spec.ts`** — ✅ robots.txt 200 (disallows /admin+/auth+/invite+/order+checkout), sitemap.xml 200 `application/xml` (`<urlset>`+`<loc>`), `/shop` = 1 canonical, `/checkout` = `noindex` + 0 canonical. → §A SEO row.
- [x] **`visitor-manuals.spec.ts`** — ✅ `/manual` (19 imgs), `/pos-manual` (4), `/labels-manual` (8), `/usage` (19): all 200 + heading + **0 broken images**. → §A manuals row.
- [ ] **Complete the `.skip` own-orders stub** — ❌ still gated: a demo sign-up (`sana…@almanar-trading.example`) is **rejected by Supabase** ("Email address … is invalid" — the `.example` domain) and storefront accounts need email confirmation. Needs a real/acceptable email + a confirmation-off seed. Stays `.skip`. → K-8 (automated half).

---

## Phase 3 — Separate browser / incognito (fresh Supabase session)

> The admin owner session shares one Supabase token, so these need a clean context (a human browser or an isolated automation profile).
>
> **Finding 2026-10-01 (Phase 0 / SET-5):** a `forceNew` browser tab in-harness opens an **isolated, unauthenticated** context (it lands on `/auth`) — i.e. an incognito-equivalent. Registering a **new demo customer** there (not the owner, so no owner password needed) may let K-8 + K-5-redeem run **in-harness** after all. Worth attempting before deferring to a human.
>
> **✅ Unblocked 2026-10-01 (Phase 3):** the owner **turned OFF email confirmation** on the Supabase project, so storefront sign-up now creates a session instantly (no email needed). K-8 ran end-to-end in the isolated/incognito tab — see below. (The earlier 429 "email rate limit exceeded" is moot now; a **Sign out** button was also added to `/onboarding`, §H #15.)

- [x] **K-8 — customer account "own orders only":** ✅ **DONE 2026-10-01.** Signed up **Amina Shaikh** (`amina.shaikh.uat@mailinator.com`) in the isolated/incognito tab → logged in instantly → `/account` Orders baseline **"No orders yet"** (0 while the shop had 7). Added **Assorted Date Ma'amoul Box** (₨1,250) → checkout (COD, Karachi, server-verified **₨1,462.50**) → **ORD-000008**. `/account` Orders then shows **only ORD-000008**, none of the owner's ORD-000001…007 → isolation proven (admin sees all 8). Findings: oversold **Gift Set (−1) blocked server-side**; online order shows **"Walk-in"** in admin (no CRM profile linked). → §K K-8 / §A account row.
- [ ] **`/r/<token>` kiosk happy-path:** **⚠️ corrected** — `/r/$token` is the **restaurant table QR self-order kiosk** (`restaurant_tables.qr_token` via `rotateTableQr`), **not** a POS receipt (receipts are print-only `window.print`). Enable QR on a table (table-QR modal → "Generate QR code") → open `/r/<token>` → kiosk menu renders. Tampered-token denial already PASS (§E). → §A `/r/$token` row.
- [x] **K-5 — loyalty redeem:** ✅ **RESOLVED-AS-FINDING 2026-10-01 (not executable — product gap).** Verified the storefront customer checkout (K-8/ORD-000008) has **no redeem control**; `store.$shopSlug.checkout.tsx` has **zero** loyalty/reward/redeem/points refs; and the token **"redeem" appears in NO component or route**. Redemption exists only as **staff-gated server fns** (`redeemForOrder`/`quoteRedemption`, behind `assertView(…,"loyalty")`) that **no UI calls** — matching the CRM/Loyalty Expansion Plan **§18** ("not wired into checkout") + **§46 matrix `[!]`**. So **EARN works** (K-5 earn ✅ prior, ORD-000006 +650 pts) but **REDEEM has no user-reachable surface** anywhere. → §K K-5 / §D11.

---

## Phase 4 — Licensing ledger (run on LIVE `goretailos.com`, UAT org only)

> **K-9 — the entire §F-LIC licensing ledger.** Sacrificial module = **Marketing**, UAT shop only; F-LIC-13 restores it to LIFETIME. **Ran on LIVE 2026-10-01 at the owner's instruction** (instead of staging) — **Trendies never touched**, `LICENSING_ENFORCED` left **OFF** (never flipped, so `[ENF]` enforcement rows stay ships-dark/ENV). **No live org is ever licensed.**

- [x] **F-LIC-0** — ✅ `/platform` loads; UAT Tenant 360 shows the 14-row Module licenses panel; `LICENSING_ENFORCED` = **OFF** ("hard enforcement is disabled").
- [ ] **F-LIC-1** — ⏭ N/A **abstained** (live-org safety); dashboard shows Trendies = "Licensed" tier (consistent w/ grandfathered); recommend the read-only SQL count for a human.
- [x] **F-LIC-2** — ✅ legacy default-allow (trial-covers-all); all 14 modules start NONE; exports work with the flag OFF (ships-dark).
- [x] **F-LIC-3** — ✅ Grant Marketing × 1 month → ACTIVE 11/1/2026 (31d); `license.granted` audited.
- [x] **F-LIC-4** — ✅ Grant Marketing × 3 months again → **stacks** to 2/1/2027 (123d, anchored at current end).
- [x] **F-LIC-5** — ✅ custom date before current end → **"earlier than the current period end — revoke first to shorten"**; row unchanged.
- [x] **F-LIC-6** — ✅ granted **CRM** custom today+10 → **EXPIRING** amber (10/12/2026, 11d); dashboard "Licenses expiring (30d)" = 1 + panel lists it.
- [ ] **F-LIC-7** — ⚠️ ENV: `license_expiry_sweep()` needs Supabase SQL (no in-harness console); the dashboard expiring panel is the UI surface.
- [ ] **F-LIC-8** — ⚠️ HARNESS-BLOCKED: **Revoke** uses `window.prompt()` for the reason → "prompt() is not supported" (same as K-7); a human completes it. Not a product bug.
- [ ] **F-LIC-9 `[ENF]`** — ⚠️ ENV/ships-dark: needs the flag ON (staging only).
- [x] **F-LIC-10** — ✅ module packs: created `zz-uat-pack` (Marketing + Loyalty) → Deactivate → Reactivate → left Deactivated.
- [ ] **F-LIC-11** — ❌→✅ **FIXED pending deploy**: pack grant threw `period_start` NOT-NULL (F-LIC-PACK-1, §H #16); fixed in `saas-licenses.server.ts`.
- [ ] **F-LIC-12** — ❌→✅ **FIXED pending deploy**: whole-org grant hit the same F-LIC-PACK-1 bug; same fix covers it.
- [x] **F-LIC-13** — ✅ **Cleanup:** Grant Marketing → **LIFETIME** (Never). Final state.
- [x] **F-LIC-14** — ✅ entitlement **DENY** `marketing` set → cleared; audited. (`data_export` not surfaced in the 14-module panel; [ENF] not verifiable flag-off.)
- [x] **F-LIC-15** — ✅ operator rails: unknown email → "No user found"; Deactivate-self blocked; role→SUPPORT-self blocked.
- [ ] **F-LIC-16** — ⚠️ ENV/ships-dark: ZZ export enforcement needs the flag ON.
- [x] **F-LIC-17** — ✅ traversal: UAT Tenant 360 → **Enter shop** → `/admin` (Al Manar UAT); audit `platform.shop_entered` · 5a00b329.
- [x] **F-LIC done** — ✅ Marketing LIFETIME, `zz-uat-pack` Deactivated, flag never flipped (OFF), **no live org touched**. Audit confirms every 10/1 action targeted shop·5a00b329 (UAT) or module_pack.

---

## Phase 5 — Single-purpose unblocks

> **Executed the non-blocker items 2026-10-01** (owner's instruction: "only test non-blocker items"). K-6 Serves ✅ done on the UAT shop; the rest are genuine blockers (harness `prompt()` / ENV infra) — recorded honestly below.

- [x] **K-6 — Domains "Serves" selector:** ✅ **DONE 2026-10-01** — added a disposable UAT test domain `zz-uat-serves-test.example` → per-domain **Serves** selector → **Restaurant website** → **persisted across reload** → AUD `shop_site.domain_assigned` (+ `domain.added`) → back to **Online store** → **removed** the test domain. ⛔ Trendies never touched. Real-DNS root serving stays ⚠️ ENV. → §D2b / §K K-6.
- [ ] **K-7 — vendor-portal submission:** ⚠️ **harness-blocked** — the secure invite link is shown via `window.prompt()` ("prompt() is not supported" in-harness); the admin invite is **Active**. A human opens the prompt link → `/vendor.rfq?token=…` → submits. → §K K-7 / §E.
- [ ] **ENV — email:** ⚠️ ENV — `/admin/emails` renders; no provider configured (honest empty state).
- [ ] **ENV — Stripe:** ⚠️ ENV — `/payment/success` + `/cancel` render; no Stripe keys (COD only).
- [ ] **ENV — integrations:** ⚠️ ENV — `/admin/integrations` hub renders with the empty state; no live providers.
- [ ] **ENV — DNS:** ⚠️ ENV — `/admin/domain-setup` instructions render; real DNS needs a registrar (the Domains add/remove/Serves flow itself ✅ works, K-6).
- [x] **Storage quota — ✅ DONE 2026-10-01 (live, UAT shop) via the Resource-limits override (no bulk upload):** set **File storage ≈ 0 MB** on the UAT Tenant 360 Resource-limits card (override is first in the `tenant_storage_cap_bytes` chain) → UAT `/admin/subscription` Usage showed **"Over Limit · 36 KB / 0 B"** → a product-image upload was **refused** by the SQL **RESTRICTIVE storage-policy backstop** (*"new row violates row-level security policy"*, HTTP 400) → **Cleared the override** → Usage back to **"36 KB / 10 GB"**. ⚠️ UX note: the friendly "Storage limit reached" pre-check didn't surface in the product-media **direct-input** path (raw RLS message instead — likely a harness artifact; a human should confirm via the "Choose file" button). ⛔ live shops never touched; override cleared. → §C storage-quota row.

---

## Definition of done

- Every box above `[x]`, **or** explicitly justified as ⚠️ ENV / ⏭ N/A in the playbook.
- Every ❌→✅ fix folded into **§H Fix Log** with root cause + change + re-run proof; the matching route row flipped to ✅.
- §G **X-07 / X-08 / X-10** green; the **live-production safety** confirmation signed off (no live org entered, modified, or licensed).
- `ZZ-` disposables deleted (ZZ trial shop may remain for license demos — note it); `LICENSING_ENFORCED` restored; UAT password rotated.

---

## VOXOPlan cross-check of this run's findings (2026-10-01)

Every GoRetailOS defect/finding above was checked against VOXOPlan (and one came back the other way):

| Finding here | VOXOPlan outcome |
|---|---|
| **F-LIC-PACK-1** — pack/whole-org grant threw `period_start` NOT-NULL | ✅ **Not present** — VOXOPlan's `grantLicenses` uses ONE unified upsert loop for module/org/pack scopes with `...(stacked ? {} : { period_start })`, so fresh rows always carry it (verified in `licenses.functions.ts`) |
| **F-WASTE-2** — `.or()` comma collision broke search | ❌ **Worse in VOXOPlan, now fixed** — the shared `likeTerm()` helper backslash-escaped commas/parens, which PostgREST's or-parser rejects outright (**proven live: PGRST100 400**), so typing a comma in ANY of ~18 search surfaces errored the list. `likeTerm` now strips `% , ( ) " \` (the proven F-WASTE-2 approach); the 2 raw sites (fleet vehicles, MCP contacts) routed through it. **Back-ported to GoRetailOS too:** `likeTerm` added to `src/lib/utils.ts` and 22 raw `.or(ilike.%${term}%)` sites across 16 files (customers, employees, POS orders, vouchers, bank rec, procurement, exports …) now use it |
| **K-7 / F-LIC-8** — `window.prompt()` blocks automation/embedded browsers | ✅ **Fixed in VOXOPlan** — its one `prompt()` (compliance evidence reject reason) replaced with an inline reason form (Textarea + Confirm/Cancel). GoRetailOS's two prompt() sites (vendor link, license revoke reason) stay as recorded here — optional follow-up |
| **§H #15** — Sign out added to `/onboarding` | ✅ **Ported** — VOXOPlan `/onboarding` now has the same top-right Sign out (wrong-account escape) |
| **F-LIC-8** — license **Revoke** reason via `window.prompt()` (harness-blocked) | ✅ **Not present** — VOXOPlan's revoke (`platform-licenses.tsx`) captures the reason in a proper controlled form (`revokeTarget`/`revokeReason` state), fully automatable. GoRetailOS's prompt stays by this tracker's decision — optional follow-up to convert it the same way |
| **F-LIC-7** — expiry sweep needs the Supabase SQL console | ✅ **Testable from the app in VOXOPlan** — `runLicenseExpirySweep` runs inside `/api/internal/outbox`, which the live 5-minute timer already drives (Round 8): grant a demo org a module expiring <14 days → within minutes the billing-manager notification + `license_notices` dedupe row + audit appear. No SQL needed |
| **F-LIC-9 / F-LIC-16** — `[ENF]` rows ships-dark | Same by design in both products: `LICENSING_ENFORCED` stays OFF in production; enforcement rows are staging-only |
| **F-LIC-14 note** — `data_export` override not surfaced in the console panel | ⚠️ **Same gap in BOTH products** — the resolver honours a `data_export` entitlement override (lets a trial-only tenant export; unit-tested in VOXOPlan), but neither console has a UI to set it; today it needs a direct DB write. Candidate small feature: a per-tenant "Allow data export" toggle next to the licenses panel, both products |
| K-3 goal-save, K-8 "Walk-in" CRM link, K-5 redeem surface, email-confirmation setting | N/A to VOXOPlan (GoRetailOS-specific features / Supabase project setting) |

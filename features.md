# Feature Register — AI-Powered Multi-Tenant Retail OS

Legend: `[x]` done and wired · `[~]` architecture/schema in place, UI or logic partial · `[ ]` not started

Status: **Phase 1 — post-audit remediation applied.** An evidence-based repository audit
(`CLAUDE_PHASE1_AUDIT.md`) found and fixed several security/accounting/concurrency defects; the
notes below reflect verified reality, not aspiration. Remaining gaps and blocked items are listed
in the audit report. Phase 2 not started.

## Phase 1

### 1. Multi-tenant, auth, users, roles, RLS
- [x] Shops (tenants) with full settings JSON (tax, inventory, pricing, purchasing, customers, notifications, receipts, AI, business hours)
- [x] Branches, registers, inventory locations per shop
- [x] Staff auth (email/password + Google), profiles auto-created
- [x] Roles in a separate `shop_users` table (owner/admin/manager/cashier/accountant/inventory/staff)
- [x] Permission-aware admin navigation (role matrix in `nav.ts`)
- [x] RLS on every tenant table via `is_shop_member` / `has_shop_role` security-definer helpers
- [x] Server-side RBAC guards (`guards.server.ts`) on every mutating server function
- [x] Shop provisioning: chart of accounts, payment methods, tax rate, doc sequences, AI permissions, storefront, policies
- [x] Permissions catalog + `role_permissions` table
- [x] Audit logging with before/after diffs + admin audit viewer
- [x] Store Data Manager (Admin → Administration → Data Manager): owner/admin per-store bulk data tools over an allowlist of 77 shop-scoped tables (Catalog / Customers & suppliers / Operations / Sales & finance / Config tiers) — structured column·operator·value conditions (eq/neq/gt/gte/lt/lte/contains/in/is-null/not-null) with a Match ALL (AND) / Match ANY (OR) toggle + live compiled-query preview, preview-count-guarded Bulk update / Bulk delete (≤50,000 rows, typed `DELETE` confirm), JSON/CSV insert/seed (`shop_id` auto-stamped, ≤1,000 rows), a recursive foreign-key dependency resolver (delete-children / set-null, nesting down the chain), and owner-only Reset (Transactional / Catalog & CRM / Everything — keeps config/setup, type `RESET`). Airtight tenant isolation (every op forced to the caller's own `shop_id`), no raw SQL (parameter-bound condition builder + identifier-validated columns), `shop_users` and `audit_logs` protected read-only, every action audited; app code only — no migration (runs server-side via the service role)
- [x] Staff invitations (create, accept, revoke) with role assignment
- [x] Sectioned shop settings UI (tax, inventory, pricing, receipts, notifications, hours, AI)
- [~] Granular per-shop permission editing UI (role matrix is code-level today)

### 2. Products, variants, SKUs, barcodes
- [x] Products, categories, brands, tax rates, variants, images, prices tables
- [x] Product create/edit with SKU, cost, opening stock, publish/unpublish; tax rate at product level
- [x] "Available for sale" toggle on products, categories and brands (`is_online`, default on) — internal-only items (raw materials, packing supplies, an internal "Packing" category, a supplier-only brand) stay usable in inventory/purchasing/admin but are hidden from both the storefront and POS product search; separate from the Active/archive flag (migration `20260901100000_online_publish_toggle.sql`)
- [x] Multi-variant builder: attribute matrix generator with per-variant price/SKU/stock
- [x] Creatable attribute combobox in the variant builder — pick an existing attribute name or type a new one and press Enter to add it inline
- [x] Barcode engine: EAN-13 with check digit, SKU-based CODE128, validation, multiple active codes per variant (unique per shop)
- [x] Scan-to-find variant lookup in products and POS
- [x] Printable barcode/price label sheets (40x30, 50x25 and more layouts)
- [x] CSV/XLSX import wizard with column mapping, validation and preview
- [x] Cost price isolated in `variant_costs` (never exposed to storefront readers)

### 3. POS, sales, payments, receipts
- [x] Standalone POS workspace shell (no admin sidebar/header), tablet and mobile optimised
- [x] Dedicated barcode scan field + fuzzy search (name/SKU/brand/category/barcode) with keyboard selection
- [x] Persistent cart on desktop/tablet, slide-in cart + sticky total on mobile
- [x] Per-line discounts, order-level discount, hold/resume sales, customer attach
- [x] Split payments: unlimited payment rows, allocated/remaining/change-due, large touch method tiles
- [x] Keyboard shortcuts (F2/F4/F6/F8/F9/Esc)
- [x] Cash sessions: open with float, cash in/out movements, close with counted cash + variance
- [x] Registers admin: create, edit, assign users
- [x] Server-side pricing and tax (frontend prices never trusted)
- [x] Payments, receipts with numbered documents
- [x] Thermal receipts: 58mm/80mm plain-text (ESC/POS-style) renderer, `@media print` fallback, reprints
- [x] Returns and refunds with restock
- [x] Inventory + double-entry accounting posted on every sale
- [x] Returns/refunds post a balanced reversing journal (fixed in audit — previously missing)
- [x] Permission-based oversell: selling into negative stock is a per-role permission (Settings → Inventory → "Who can oversell (negative stock)", default owner/admin/manager) — non-permitted staff blocked at the till and server-side; anonymous online checkout still follows the shop-wide "Allow negative stock" toggle
- [x] Credit / udhaar (pay-later) sales: enable in Settings → Customers & credit + allowed roles; the POS "Credit (pay later)" tender completes with partial/zero payment, requires a customer, books the unpaid balance to Accounts Receivable (1150) and enforces the customer's credit limit (manager override) — reuses the existing `record_pos_sale` RPC
- [x] Sales screen (Admin → Sales → Sales): lists counter/online/AI sales with filters + pagination, a full sale detail (line items, tenders, totals breakdown, paid vs outstanding, cashier, register, customer, returns) and a Record payment action to settle credit/partial sales
- [ ] Native ESC/POS device driver (browser/plain-text printing only; ESC/POS bridge adapter present)

### 4. Inventory, purchasing, suppliers
- [x] Inventory balances, movements log, adjustments
- [x] Stock transfers between locations (draft → send → receive)
- [x] Stock counts with variance review and posting
- [x] Suppliers, purchase orders, goods receiving (updates stock + average cost)
- [x] Multi-product purchase orders: a single PO holds many line items (add/remove line rows, per-line quantity + unit cost, live running total) — goods receiving already iterates every line, so receiving is unchanged
- [x] Expandable PO rows: clicking a purchase order reveals an inline detail table (Product · Ordered · Received · Unit cost · Line total); mobile shows the same as card rows
- [x] Edit or cancel a purchase order — change supplier and replace its line items (PO number and status preserved) or cancel — but only while nothing has been received (status not received/cancelled and every line's received qty = 0); once any goods are received the PO is locked from edit/cancel (the UI shows a short note), because reversing posted stock/costs is out of scope for this batch
- [x] Supplier invoices and supplier payments with balance tracking
- [x] Low stock detection, reorder points and alerts

### 5. Accounting, finance, reports
- [x] Chart of accounts, journal entries, double-entry lines
- [x] Automatic posting for sales, returns, purchases, expenses
- [x] Expenses + expense categories
- [x] Trial balance and general ledger
- [x] Profit & loss, balance sheet, cash flow statement
- [x] AR / AP ageing reports
- [x] Dedicated report suites (mirroring the Accounting page — report picker + From/To range + Print + Download CSV, role-gated, server-side aggregates over existing tables): Sales Reports (summary, by day/product/category/brand/payment/channel/cashier/register/branch, tax, returns), Purchase Reports (PO status, supplier spend, by product, goods receipts, outstanding POs, AP ageing, cost history), Inventory Reports (valuation, by location, low stock, dead stock, movements, adjustments, turnover)

### 6. Ecommerce storefront (own Tailwind design system)
- [x] Per-shop custom domains: connect a shop's own domain(s) at Storefront → Domains (backed by a `shop_domains` table + host→shop resolver + router URL rewrite), serving the storefront at clean root URLs (`/`, `/product/<slug>`, `/cart`, `/checkout`) with no `/store/<slug>/` prefix; one domain = one shop (globally unique), unverified/unknown hosts never resolve, secondary domains 301-redirect to the shop's primary. Setup: add domain → DNS (A → 185.158.133.1 + `_lovable` TXT) → verify + SSL → mark verified → make primary. Merchant guide `CONNECT_DOMAIN_NAMECHEAP.md`, in-app guide `/admin/domain-setup`
- [x] Storefront design system in `src/components/storefront/ui.tsx` — buttons, inputs, cards, sheets, badges, product grid, skeletons, empty states — driven by the shop's `--theme-*` palette
- [x] Zenis theme CSS removed entirely: no Bootstrap, no jQuery-plugin stylesheets, no Font Awesome Pro dependency (~1MB less CSS per page load)
- [x] Per-shop colors, logo, favicon, tagline, fonts, announcement bar, hero, social links
- [x] Full route set: home, shop, category, product, cart, checkout, wishlist, compare, flash deals, track order, auth, account, vendor profile, about, contact, policy pages, payment success/cancel — every one rebuilt on the design system
- [x] Storefront CMS: banners, sections, navigation menus and custom pages editable in admin
- [x] Theme customization panel (colors, typography, layout, homepage blocks) with live preview
- [x] Products, prices, stock, categories, brands, policies, business hours all from the database
- [x] Multi-currency formatting, multi-language (en/ar/ur/hi) with RTL and a storefront locale switcher
- [x] Guest and signed-in checkout creating real orders + inventory + ledger entries
- [x] Customer order tracking by order number + contact
- [x] Customer portal: multi-shop customer record (`shop_id` + `user_id`), profile, saved addresses, order history, wishlist
- [x] Responsive at 375 / 390 / 412 / 768 / 1024 / 1440 — zero horizontal overflow on every page, verified by the Playwright suite
- [x] Uploaded images (shop logo, brand logos) degrade to a text fallback instead of a broken-image glyph
- [~] Product reviews and ratings (render + submit; no moderation UI)
- [ ] Homepage variant selector (index … index-5) — removed with the theme; the homepage now has one responsive layout whose section order lives in `store.$shopSlug.index.tsx`

### 7. AI Salesman + AI Business Assistant
- [x] AI Salesman chat widget on every storefront page (guest friendly)
- [x] Salesman tools: catalog search, stock check, price check, recommendations, cart proposals, shop info & policies
- [x] Salesman catalog search filters by price (min/max), category and on-sale and returns live stock + was-price; a `product_reviews` tool; `shop_info` also returns accepted payment methods + the category list; always returns a text answer (fixed the "I could not complete that request" dead-end)
- [x] AI Business Assistant in admin: sales summary + period comparison, product performance & margins, inventory/reorder status, financial summary, purchasing summary, alerts
- [x] Business analysis on real signals (`ai-insights.server.ts`): low stock, reorder needs, dead stock, receivables, margins
- [x] Per-shop AI permission matrix (disabled / approval / automatic) enforced server-side
- [x] AI action execution end to end, with approval queue UI (approve/reject, result + error recorded)
- [x] Assistant approve→execute→feedback loop wired: pending approvals link to the chat conversation (`conversation_id`), a `lookup_ids` tool resolves supplier/product/invoice names & SKUs to real IDs so write-action payloads validate, the advertised action list corrected (removed the non-existent `issue_refund`; added `record_payment`, `generate_report`, `create_supplier_invoice`, `draft_message`), and missing `ai_permissions` capability rows self-heal on use — on Approve the action runs and the result posts back into the chat (role limits still apply)
- [x] Proactive assistance: morning briefing and end-of-day summary
- [x] Global ⌘K command centre with inline assistant answers
- [x] Conversation history + context memory per conversation
- [x] Knowledge base: document ingestion, chunking, `tsvector` + GIN full-text retrieval, tenant-isolated, cited answers
- [x] Durable business memory (`ai_business_memory`) with manager approval before use
- [x] Alert engine (low stock, new order, overdue receivable) feeding dashboard + AI
- [ ] Voice input/output

### 8. Admin portal design system
- [x] Isolated admin token layer under `[data-app="admin"]` — indigo/violet oklch palette, glass, gradients, elevation
- [x] Depth system: `admin-panel`, `admin-raised`, `admin-sunken`, `admin-elevate`, depth-aware shadows — no flat surfaces
- [x] Light and dark themes with header toggle
- [x] Zero Zenis/Bootstrap leakage into admin; storefront CSS scoped to its layout route
- [x] Fixed collapsible sidebar with grouped, role-filtered navigation; mobile drawer
- [x] Header: shop switcher, branch switcher, global search, notifications, AI shortcut, profile, theme toggle
- [x] Management dashboard: KPI cards with trends (sales, orders, profit, margin, inventory value, low stock, receivables, payables), sales area chart, category donut, profitability bars, AI insight panel, alert centre
- [x] Reusable primitives library (`AdminUI.tsx`) used across admin pages, incl. touch button variants
- [x] Every management table has a `MobileCards` layout; no horizontal overflow at 375px

### Offline / PWA
- [x] Manifest + icons (installable)
- [x] Service worker: app shell + asset caching, network-first navigation
- [x] IndexedDB (v2) catalog + barcode index cache, pending-sale queue, conflicts store
- [x] Device identity + per-device monotonic local sequence for offline ordering
- [x] POS falls back to the offline queue when the network fails or checkout errors
- [x] Automatic background sync with duplicate protection (`local_uuid` unique per shop)
- [x] Pricing/tax versioning on shops with triggers; offline sales carry version + client snapshot
- [x] Offline sales capture a client price snapshot at the till and send it on sync (fixed in audit — snapshot was previously never populated)
- [x] Server-authoritative reconciliation on sync; variances recorded in `pos_sync_conflicts`
- [x] Sync conflict console at `/admin/sync` (accept / reject / retry, owner-admin-manager only); retry re-posts rejected sales from their stored snapshot
- [x] Offline/sync status surfaced in the POS session bar

## Phase 2 (not started, schema-ready)
- [ ] CRM, loyalty, store credit, segmentation
- [ ] Marketing, social media, WhatsApp/SMS/email campaigns
- [ ] Advanced analytics, forecasting, automation, SaaS billing

### Iteration 6
- [x] Printer abstraction layer with browser + ESC/POS bridge adapters and automatic fallback
- [x] Per-device printer configuration UI with connection test
- [x] Settings: pricing & discount limits, purchasing defaults, document numbering, business hours

### Iteration 7 — audit remediation (see `CLAUDE_PHASE1_AUDIT.md`)
- [x] P0: public `quoteCart` no longer returns `unit_cost` — cost price cannot leak to the storefront
- [x] P1: public AI Salesman `shop_info` no longer returns the knowledge base or raw `shops.settings`
- [x] P1: returns/refunds post a balanced reversing journal entry (revenue/tax/COGS/cash)
- [x] P1: offline sales capture and sync a client price snapshot → reconciliation actually runs
- [x] P1: sales post with best-effort rollback so a failed step can never leave a half-posted order
- [x] P1/P2: atomic `next_document_number` + `apply_inventory_delta` RPCs (no numbering/stock races)
- [x] P1: broad `is_shop_member` write policies on sales/catalog/inventory tables replaced with role-scoped policies; financial doc tables are server-write-only
- [x] P2: POS checkout/refund role-gated; AI assistant financial/cost tools gated to finance roles; online COD posts to A/R not cash
- [x] P2: automated test suite (`npm test`) — pricing/tax, barcode, localization, RBAC guards
- [x] P3: `allow_negative_stock` enforced for live POS sales; knowledge FTS analyzer fixed; theme CSS sanitized; `.env` untracked

## UI Responsiveness & Interaction Stability
- [x] Proper error messages app-wide: a `getErrorMessage()` helper (`src/lib/utils.ts`) surfaces the real backend reason (duplicate SKU, validation, permission, …) instead of a generic "Could not save…" — applied at ~76 call sites across ~36 files (admin routes, storefront checkout/product/payment, POS, printing, import/sync, AI). TanStack Start server-function errors arrive on the client as plain objects, not `Error` instances, so the old `err instanceof Error ? err.message : "…"` pattern always fell through to the generic fallback
- [x] Numeric inputs (cost, price, sale price, weight, opening stock, PO quantities) select their contents on focus, so a leading "0" no longer sticks when typing
- [x] Async action buttons provide immediate loading feedback (spinner + disabled)
- [x] Duplicate async submissions prevented (POS Complete sale, refunds, goods receiving, invoices/payments, stock-count approve, inventory adjust, expenses, AI/storefront/team actions)
- [x] Button pressed/active + focus-visible states (existing `btn` primitive; `Spinner`/`LoadingButton` added to `AdminUI.tsx`)
- [x] Dashboard loads with a full skeleton mirroring the real layout (KPI grid, panel headers, chart heights) — no more "dancing"
- [x] Skeleton/spinner loading states work in light + dark themes
- [~] Loading states applied to the priority async buttons; a few low-traffic buttons still to convert

## Data Tables — Search & Server-Side Data Loading
- [x] Orders table: server-side search (order #/contact) + status/payment/channel/date filters + pagination & page size
- [x] Inventory movements: server-side type/date filters + pagination & page size
- [x] Reusable `Pagination` primitive + `keepPreviousData` so rows never blank while filtering/paging
- [~] Smaller config/master-data + team tables — client-side acceptable for current volume
- [ ] Deferred high-volume/entangled tables (Products, accounting aggregate reports, purchasing PO/invoice tables, sync conflicts, AI approvals) — see `handoff.md` → "Deferred — High Impact"

## Storefront UI/UX revamp (Iteration 9)

### Design system
- [x] `src/components/storefront/ui.tsx` — `SFContainer`, `SFSection`, `SFPageHeader`, `SFBreadcrumb`, `SFCard`, `SFButton`/`sfBtn`, `SFIconButton`, `SFInput`/`SFSelect`/`SFTextarea`/`SFField`, `SFQuantity`, `SFBadge`, `SFSectionTitle`, `SFPrice`, `SFProductGrid`, `SFSkeleton`, `SFProductCardSkeleton`, `SFEmpty`, `SFAlert`, `SFSheet`, `SFImageOrText`
- [x] Every surface driven by `--theme-*` variables, so the admin theme editor re-brands the whole storefront
- [x] Logical properties (`ms-`/`me-`/`start-`/`end-`) throughout — Arabic/Urdu RTL mirrors with no second stylesheet
- [x] Minimum 36px tap targets on every interactive control, enforced by test

### Theme removal
- [x] All 16 Zenis stylesheets dropped from the storefront layout route
- [x] Bootstrap grid, `.offcanvas`, `.theme_btn`, `.common_btn`, `wow`/`animate.css`, slick, select2, venobox, pwstabs, nice-select, range_slider — all references removed from source
- [x] Font Awesome icon classes replaced with `lucide-react` components + an inline Google brand mark
- [x] `useWowReveal` deleted — it left 76 of 82 elements at `visibility:hidden` until scrolled into view
- [x] `public/theme/assets/images` retained (category/product fallback art is referenced by path)

### Rebuilt surfaces
- [x] `StoreHeader` — sticky bar, category mega-menu, mini-cart sheet, mobile drawer, mobile search toggle, locale switcher
- [x] `StoreFooter` — brand/categories/quick-links/contact columns, business hours, lucide social icons
- [x] `HomeBanner` / `HomeFeatures` — hero slider with arrows, dots, autoplay-pause-on-hover and a contrast scrim; desktop category rail; promo panel
- [x] `HomeProducts` — flash sale (+countdown), category tiles, trending with category pills, best sellers, new arrivals, favourites, brands, newsletter
- [x] `ProductCard`, cart, checkout, shop, category, product detail, wishlist, compare, track order, account, sign-in/up, forgot-password, about, contact, vendor, policy pages, payment success/cancel
- [x] `AiSalesman` — full-width bottom sheet on mobile, typing indicator, auto-scroll
- [x] `ScrollButton` — Tailwind progress ring, positioned clear of the AI launcher

### Fixed defects
- [x] 49px horizontal overflow at 375px (Bootstrap `.offcanvas-end` stayed in document flow while off-screen)
- [x] Shop page rendered a fixed 240px filter sidebar on phones (`minmax(0,240px) 1fr`, no media query) — now a filter sheet
- [x] Home page content invisible on load (wow-reveal `visibility:hidden` + 3s failsafe)
- [x] Broken-image glyphs for shop/brand logos when an upload fails to load (now a text fallback; also handles images that fail before React hydrates)
- [x] Hero pager dots were 8×8 tap targets — now 36px hit areas around an 8px dot
- [x] Desktop page height cut from 12,904px to ~7,200px

### Playwright
- [x] `playwright.config.ts` — 6 viewport projects (375/390/412/768/1024/1440), drives installed Edge via `channel: "msedge"` so no browser download is needed
- [x] `tests/e2e/storefront-responsive.spec.ts` — per-page overflow, content-rendered and tap-target assertions, plus navigation, cart flow and theme-isolation tests
- [x] `tests/e2e/storefront.helpers.ts` — overflow/tap-target/invisible-element probes that name the offending elements on failure
- [x] `npm run test:e2e` / `test:responsive` / `test:e2e:ui`; runs against an existing dev server by default, `PW_START_SERVER=1` to boot one

## POS responsiveness + render performance (Iteration 10)

The POS looked right but felt wrong: a visible stutter on every keystroke, and a layout that broke
below tablet width. Both were measured with Playwright before and after, against the real
626-variant demo catalog rather than an empty shop.

### Main-thread cost (4x CPU throttle, `pos-1440`)

| Interaction | Before | After |
| --- | --- | --- |
| Type in search — total blocking | 1149 ms | 117 ms |
| Type in search — worst task | 245 ms | 111 ms |
| Type in scan field — total blocking | 1287 ms | 8 ms |
| Type in scan field — long tasks | 11 | 1 |
| Add 6 items to cart — worst task | 216 ms | 105 ms |
| Idle 16 s — long tasks | 0 | 0 |

### Causes fixed
- [x] Keyboard-shortcut `useEffect` had **no dependency array** — it removed and re-added a
      `window` keydown listener on every single render. Now bound once, handler kept fresh via a ref.
- [x] The 5-second offline-sync poll always called `setRecent` with a freshly built array, so the
      whole 1,300-line component re-rendered four times a minute forever. It now returns the
      previous state when nothing actually changed.
- [x] The product grid and the cart were plain `const grid = (...)` / `const cart = (...)` JSX
      inside the page component, so both were rebuilt on every state change anywhere — including
      each character typed into the scan field, which filters nothing. Both are now `memo`
      components fed by stable `useCallback`/`useMemo` props.
- [x] Search lowercased six fields on ~626 items on every keystroke (~4,000 string allocations per
      character). A lowercased haystack is now built once per catalog load.
- [x] Search filtering runs through `useDeferredValue`, so the caret commits at high priority and
      the grid re-renders as interruptible work.
- [x] The filter loop stops at 60 matches instead of scanning the whole catalog and slicing.
- [x] `items` fell back to a fresh `[]` literal while loading, invalidating every downstream memo.
- [x] `activeRegister` and all cart/payment totals were recomputed unmemoised each render.

### Layout fixed
- [x] Root used `h-[calc(100svh-3.5rem)]`, subtracting a header that is its own child — and that
      header wraps. Inside `PosShell`'s clipped `100dvh` box the extra rows pushed the cart bar and
      payment sheet off-screen. Now `flex-1 min-h-0`.
- [x] Header wrapped to three rows (up to 200 px of an 812 px phone) — register and cash-session
      controls now collapse behind a disclosure below `lg`.
- [x] Catalog was `sm:grid-cols-3` at every width above 640, so at 768 the cart left it 416 px split
      into 128 px tiles. Now steps back to two columns in the `md` band, and the cart width steps
      19rem → 21rem → 26rem instead of jumping to 22rem.
- [x] Touch targets: category chips, header controls, cart controls, modal close buttons and the
      order-discount field were 32–36 px. All POS controls are now ≥ 40 px (`btn.pos`,
      `btn.posGhost`, `posInput`).
- [x] Payment sheet used `max-h-[92vh]`; on mobile `vh` exceeds the visible viewport. Now `dvh`.
- [x] The cart renders twice below `md` (persistent aside + slide-in sheet), so its order-discount
      input shared one `id`. Now label-wrapped with no `id`, and F6 targets the visible copy.

### Correctness fixed along the way
- [x] The sync poll overwrote each pending sale's stored receipt, permanently disabling "Reprint"
      for offline sales. It now merges instead of replacing.
- [x] The grid showed "No products yet — add products to start selling" while the catalog was still
      loading. It now shows a skeleton.

### Playwright
- [x] `tests/e2e/auth.setup.ts` — signs a staff user in once and saves storage state. Credentials
      come from `PW_EMAIL` / `PW_PASSWORD` only; `tests/.auth/` is gitignored.
- [x] `PW_SHOP_ID` selects which shop the POS opens, so measurements run against a real catalog
      instead of whichever shop the account happens to list first.
- [x] `tests/e2e/pos-responsive.spec.ts` — layout across 6 viewports + main-thread measurements
      (long tasks / total blocking time under a 4x CPU throttle). The sharpest assertion is a
      *ratio*: scan-field typing and search typing are measured in the same page at the same
      moment, so machine speed and dev-server load cancel out. Scanning must cost a fraction of
      searching; before the fix the two were equal (1287 ms vs 1149 ms), which is what proved every
      keystroke was re-rendering the whole workspace. It now measures ~26 ms against ~134 ms.
- [x] `tests/e2e/pos-behaviour.spec.ts` — guards the refactor: add/increment, quantity controls,
      F8 hold + resume, F2/F4/F6/F9/Escape shortcuts, search filtering, order discount. It never
      completes a sale, so no test run writes an order into a real shop.
- [x] `npm run test:pos`

## Admin portal desktop UI consistency (Iteration 11)

The admin portal is a desktop tool, so responsiveness was explicitly out of scope. What was in
scope: form fields that didn't share a baseline, controls at four different heights, elements
colliding, and the render churn behind the "jerks".

### The alignment defect

`Field` renders its label as a CSS grid, and a grid item stretches to its row's height by default.
So in a row where one field carried a hint and its neighbour didn't, the hint-less field's rows
stretched to fill the taller box and its input drifted *down* — measured on `/admin/settings` as
`Shop name` at y=305 against `Store address` at y=295. One `content-start` on `labelClass` pins
every control in a row to the same offset regardless of what hangs below it.

### Two competing field components

`admin.products.tsx` exported its own `Input` — a `text-sm` label over a 38px control — which
`/admin/purchasing` and `/admin/accounting` imported. Any form mixing it with a plain `Field` got
two label sizes and two control heights in one column. It now delegates to `Field` + `inputClass`.

### One scale, two sizes

The admin now has exactly two control heights: **40px** for form controls (`inputClass`) and
**32px** for the dense furniture around a table (`inputClassCompact` — pagers, inline row editors),
matched to `btn.small`. Everything that sat between them was normalised:

| Where | Was | Now |
| --- | --- | --- |
| Pager "rows per page" | 36px beside 32px buttons | 32px |
| `/admin/ai` chat composer | 44px | 40px |
| `/admin/ai` permission selects | 31px | 32px |
| `/admin/ai` knowledge + memory inputs | 38px, ad-hoc classes | 40px |
| Variant builder matrix | 30–31px | 32px |
| purchasing / accounting `Input` | 38px | 40px |

`/admin/ai` had no design-system usage at all — every control was a bespoke class string. 22 of
them now map onto `btn.*`, `inputClass`, `inputClassCompact` or `textareaClass`, and its three
textareas no longer inherit `inputClass`'s `h-10` (which was crushing an 8-row box to one line).

### Render churn fixed

- **`MobileCard` rendered a `<button>` wrapper and then put its `actions` buttons inside it.** A
  button cannot contain a button, so React reported a hydration mismatch and threw the
  server-rendered tree away to re-render on the client — a visible flash on every page using these
  cards. Only the summary is the button now.
- **`/admin/orders` keyed the wrong element.** `rows.map()` returned a `<>…</>` with the `key` on an
  inner `<tr>`, so React never saw it and remounted every row on each render.
- **`SyncStatus` set state from an async IndexedDB read** with no mounted guard. It mounts on every
  admin page; under StrictMode's double-invoked effects the first run's continuation could land
  during the second mount, warning and scheduling an extra render pass. This accounted for 8 of the
  9 console-error failures in the audit.

### Playwright

- `tests/e2e/admin-ui.spec.ts` — all 27 admin pages at 1440: console errors, field-baseline
  alignment, control scale, overlap, panel overflow.
- `tests/e2e/admin-forms.spec.ts` — opens the create form on 14 pages and re-applies the alignment,
  scale and overlap checks, because most admin forms are behind a "New …" action and the static
  pass never sees them.
- `tests/e2e/admin.helpers.ts` — the probes. The overlap probe skips elements clipped by a
  scrollable ancestor: an element scrolled below the fold of an `overflow-y-auto` list still
  reports its real page coordinates and looks like it is lying on top of the next section. Two of
  the first run's "overlaps" were that artifact, not defects.
- `npm run test:admin`

## Admin tabs, toggles and the variant matrix (Iteration 12)

Three defects, three shared primitives.

**Tabs stopped moving.** The storefront editor gave inactive tabs a 1px border and the active tab
none, so selecting a tab shrank it by 2px and reflowed the row; `/admin/ai` swapped in
`font-semibold`, which made the selected label wider. Both now use a shared `Tabs` component whose
geometry is fixed: the underline is always present (transparent when inactive) and the weight never
changes, so only colour differs between states. Tab labels are explicit, because `capitalize` on the
raw key rendered "Faq".

**One boolean control.** Raw `<input type="checkbox">` appeared 23 times, sized anywhere from an
unstyled browser default to `size-3.5` beside a 40px search field. They are now:

| Primitive | Use | Sites |
| --- | --- | --- |
| `Toggle` | on/off state — the admin's only boolean control | 14 |
| `Checkbox` | picking items out of a set | 7 |

The split is deliberate. A 44px switch per table row would dominate the column and read as seven
independent settings rather than one multi-select, so label-sheet row selection, the field picker,
stock-count categories, variant attribute values and onboarding locales stay checkboxes — properly
styled ones. Zero raw checkboxes remain outside the `Checkbox` primitive itself.

**The variant matrix fits again.** `FormSection` lays a 14rem heading column beside its content,
which is right for a column of fields and wrong for a full-width table — the variant matrix was
being crushed into what was left. A `wide` prop stacks the heading above instead; "Variants, pricing
& stock" uses it.

### A coverage regression this iteration caused, and caught

Adopting `Tabs` changed the storefront tab buttons from `role="button"` to `role="tab"`. The tab
spec queried `getByRole("button")`, found nothing, and did what it was told: `test.skip`. Seven tab
audits silently stopped running and the suite still reported "0 failed" — the pass count dropped
from 190 to 185 while skips rose from 3 to 10, which is the only reason it was noticed.

The selector now matches either role, and each tabbed page has a **reachability guard** that asserts
every expected tab is findable and fails if it is not. A skip can no longer masquerade as a pass.

## Every create/edit form is a modal (Iteration 13)

All 17 create/edit surfaces now open in one shared `FormModal`: 10 through `MasterDataPage`
(categories, brands, attributes, tax, locations, branches, accounts, expense-categories, payments,
suppliers), 4 on purchasing (supplier, purchase order, invoice, payment), and one each on products,
transfers, stock-counts, registers, team and accounting.

`FormModal` owns the parts that were previously per-page: header, padding, a scrolling body with a
pinned footer so Save never scrolls out of reach, Escape and backdrop-click to close, body scroll
lock, and `max-h-[90dvh]` rather than `vh` so browser chrome cannot push it off screen.

Width is explicit rather than uniform. A category has three fields; the product form embeds the
variant matrix and an image uploader. Four tiers — `sm` / `md` / `lg` / `xl` — mean products gets
`max-w-6xl` and a category gets `max-w-md`, instead of everything scrolling sideways inside a box
built for a name and a checkbox.

**Four forms had no trigger at all.** Registers, team, purchasing and accounting simply kept a form
sitting on the page, which is why the create-form audit had never reached them — it looks for a
"New …" action and skipped when it found none. They now have one, and the audit covers them:
create-form tests went from 39 passed / 3 skipped to **52 passed / 0 skipped**.

## Admin dark theme and the phone-width top bar (Iteration 14)

### The top bar was white in dark mode, and so was every panel

`@theme inline` maps `--color-card: var(--card)` and Tailwind emits that at `:root`, where the
`var()` is substituted once. `[data-app="admin"].dark` overrides `--card`, but `--color-card` keeps
the value it resolved to on `:root`. Measured live:

```
--card       : oklch(20.7% .038 268)   <- dark, correct
--color-card : oklch(100% 0 0)         <- still white
header bg    : oklab(1 0 0 / 0.78)     <- white at 78%
```

Tailwind's own utilities were unaffected — `@theme inline` compiles `bg-card` to `var(--card)` — so
`bg-card` elements went dark while the hand-written CSS around them stayed white. Three `@utility`
blocks read `var(--color-*)`: `admin-glass` (the top bar), `admin-panel` (**every panel, modal and
dropdown**) and `admin-scroll`. All now read the tokens the admin scope actually overrides.

### The top bar at 375px

The admin is a desktop tool and is audited at 1440 only — but the top bar is the one piece people
reach for on a phone, and all three of its controls were broken there:

| Defect | Before | After |
| --- | --- | --- |
| Notifications opened off-screen | panel at `left: -57` | `12..363` inside a 375 viewport |
| Shop switcher sat on top of search | shop `56..218`, search `195..235` | `56..130` / `198..235` |
| Branch picker absent entirely | `hidden md:block` | visible, `138..190` |

The overlap cause is worth remembering: the shop switcher's *wrapper* was `min-w-0` so it shrank
under flex pressure, but the button inside kept its intrinsic width and spilled out of it.
Constraining the button is what makes the truncation happen. Notifications render as a full-width
sheet below `sm` and revert to an anchored dropdown from `sm` up; the branch picker collapses to its
icon below `sm` so it costs 36px instead of 119px.

Both fixes carry regression tests (`admin-theme.spec.ts`, `admin-topbar.spec.ts`) that were verified
to fail against the pre-fix code and pass after it.

# GoRetailOS — Playwright QA Issues Log

This file logs every meaningful problem encountered during Playwright implementation and execution.
Only issues actually encountered in GoRetailOS are recorded.

---

## QAISSUE-001

- **Stage:** Stage 2 (P0 Smoke)
- **Related test ID:** SMOKE-002 (POS loads)
- **Classification:** TEST BUG
- **Problem:** The POS page has no `<main>` element (it is a full-bleed workspace with no admin shell), so the `gotoAdmin` helper's `expect(page.locator("main").first()).toBeVisible()` timed out.
- **Root Cause:** The POS route (`/admin/pos`) renders inside `PosShell` which has no `<main>`; the admin shell (`AdminLayout`) is bypassed for POS.
- **Resolution:** Updated `gotoAdmin` in `qa.helpers.ts` to wait for the POS scan field (`getByPlaceholder(/Scan barcode/i)`) when the path starts with `/admin/pos`.
- **Files Changed:** `tests/e2e/qa.helpers.ts`
- **Verification:** Re-run SMOKE-002.
- **Lesson:** POS is a distinct layout; do not assume every admin page has a `<main>`.

---

## QAISSUE-002

- **Stage:** Stage 2 (P0 Smoke)
- **Related test ID:** SMOKE-005 (Purchasing loads)
- **Classification:** TEST BUG
- **Problem:** `getByRole('heading', { name: /Purchasing|Purchase Orders/i })` resolved to 2 elements (strict mode violation): the h1 "Purchasing & suppliers" and the h2 "Purchase orders".
- **Root Cause:** The regex matched both the page title and a panel heading.
- **Resolution:** Used `exact: true` with the actual h1 text "Purchasing & suppliers".
- **Files Changed:** `tests/e2e/owner-smoke.spec.ts`
- **Verification:** Re-run SMOKE-005.
- **Lesson:** Assert on the actual rendered heading text, not a route-derived guess.

---

## QAISSUE-003

- **Stage:** Stage 2 (P0 Smoke)
- **Related test ID:** SMOKE-008 (Customers loads)
- **Classification:** TEST BUG
- **Problem:** `getByRole('heading', { name: 'Customers' })` resolved to 2 elements (strict mode violation): the h1 "Customers" and the h2 "All customers".
- **Root Cause:** The page has both a page title and a panel heading containing "Customers".
- **Resolution:** Used `exact: true` with the h1 "Customers".
- **Files Changed:** `tests/e2e/owner-smoke.spec.ts`
- **Verification:** Re-run SMOKE-008.
- **Lesson:** Use `exact: true` when a heading name is a substring of another heading.

---

## QAISSUE-004

- **Stage:** Stage 6 (POS + Sales)
- **Related test ID:** POS-031 (discount cap regression)
- **Classification:** TEST BUG
- **Problem:** The POS "Opening float" input is disabled until a register is selected. The test tried to fill the disabled input and timed out.
- **Root Cause:** The POS requires a register to be selected before a cash session can be opened. The opening float input is `disabled` until a register is chosen.
- **Resolution:** Updated the test to select a register first (click the Register button, choose the first register option) before filling the opening float.
- **Files Changed:** `tests/e2e/owner-pos-financial.spec.ts`
- **Verification:** Re-run POS-031.
- **Lesson:** The POS register selection gates the cash-session controls; always select a register before interacting with the session bar.

---

## QAISSUE-005

- **Stage:** Stage 10 (Storefront)
- **Related test ID:** STORE-010 (invalid shop slug)
- **Classification:** TEST BUG
- **Problem:** `getByText(/Store unavailable|Store does not exist/i)` resolved to 2 elements (strict mode violation): the h1 "Store unavailable" and the p "This store does not exist."
- **Root Cause:** The regex matched both the heading and the paragraph.
- **Resolution:** Used `getByRole('heading', { name: 'Store unavailable', exact: true })`.
- **Files Changed:** `tests/e2e/visitor-storefront-security.spec.ts`
- **Verification:** Re-run STORE-010.
- **Lesson:** Use role-based selectors with `exact: true` when text overlaps.

---

## QAISSUE-006

- **Stage:** Stage 10 (Storefront)
- **Related test ID:** STORE-008 (compare add/remove)
- **Classification:** TEST BUG
- **Problem:** `getByText('Bag 0698-1').first()` matched a hidden mobile card on the compare page (the desktop table is the visible layout at 1440px).
- **Root Cause:** The compare page renders both a mobile card layout (`sm:hidden`) and a desktop table (`hidden sm:block`). At 1440px the mobile cards are hidden, so `.first()` matched the hidden element.
- **Resolution:** Used `.last()` to target the visible desktop table element.
- **Files Changed:** `tests/e2e/visitor-storefront-cart.spec.ts`
- **Verification:** Re-run STORE-008.
- **Lesson:** Storefront pages have responsive dual layouts (mobile cards + desktop table); target the visible layout for the viewport.

---
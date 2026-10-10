You are working inside the **GoRetailOS / Retail OS repository**.

Before implementing Playwright or running the test plan, I want you to proactively review and apply lessons learned from a previous project's Playwright QA execution.

The objective is NOT to blindly copy the previous project's test code.

The objective is:

**Understand the previous problems → inspect how GoRetailOS differs → proactively avoid the same failures → fix applicable setup/test issues in GoRetailOS → document any new GoRetailOS-specific lessons.**

Use the previous QA problem/resolution log provided to you as a reference.

---

# 1. FIRST: READ THE PREVIOUS QA LESSONS

Carefully study the previous project's Playwright problems and resolutions.

Extract lessons related to:

* VS Code terminal approval prompts
* PowerShell execution policy
* `npm.cmd` / `npx.cmd`
* Playwright installation
* browser installation
* dotenv
* `.env.test`
* global setup
* storageState
* absolute URLs
* baseURL
* test project configuration
* authenticated personas
* unauthenticated visitor project
* role-specific projects
* platform-level vs tenant-level personas
* first-run Quick Tour/onboarding modals
* localStorage initialization
* Radix/modal accessibility behaviour
* semantic selectors
* actual rendered heading text
* documentation vs live RBAC grants
* tenant-specific test scoping
* visitor-only test scoping
* authenticated-only test scoping
* avoiding overly fragile smoke tests
* full regression project multiplication
* pass/fail/skip interpretation

Do NOT assume every previous issue exists in GoRetailOS.

Check the GoRetailOS code first.

---

# 2. INSPECT GORETAILOS BEFORE ADDING PLAYWRIGHT

Read:

* `AGENTS.md`
* `README.md`
* `features.md`
* `handoff.md`
* `QUICKSTART.md`
* `usage.md`
* `USER_MANUAL.md`
* `POS_MANUAL.md`
* `TEST_CASES.md`
* `package.json`
* existing test folders
* existing Playwright config if present
* auth code
* onboarding
* Quick Tour
* role/permission implementation
* shop switching
* route guards
* environment loading
* local storage/session handling

Also inspect current database migrations relevant to:

* users
* shops
* `shop_users`
* roles
* permissions
* role permissions

The actual current code and migrations are authoritative.

Do NOT trust an older documentation matrix if live grants differ.

---

# 3. CHECK WHETHER PLAYWRIGHT ALREADY EXISTS

Before installing anything:

Check for:

* `@playwright/test`
* `playwright.config.ts`
* `tests/`
* `e2e/`
* existing browsers
* existing `.env.test`
* existing auth setup
* existing storageState files
* package scripts

Do NOT recreate an existing config from scratch unnecessarily.

Edit existing files where possible.

Do NOT duplicate dependencies.

---

# 4. WINDOWS / POWERSHELL EXECUTION RULES

This development machine may use Windows PowerShell.

If standard commands such as:

`npm`
or
`npx`

fail because PowerShell blocks unsigned `.ps1` scripts, use:

`npm.cmd`

and:

`npx.cmd`

instead.

Examples:

`npm.cmd install`

`npm.cmd install -D @playwright/test`

`npx.cmd playwright install chromium`

`npx.cmd playwright test`

Do not change system-wide execution policy merely to make tests run.

Prefer the `.cmd` shim.

---

# 5. VS CODE TERMINAL APPROVAL PROMPTS

If VS Code intercepts terminal commands with something like:

`[V] Never run [D] Do not run [R] Run once [A] Always run`

do not repeatedly submit commands into a blocked terminal.

Use the available terminal mechanism to obtain the terminal/session ID and respond to the pending prompt.

If the environment allows an "Always run" choice and the user has configured that intent, use it appropriately.

Do not mistake a terminal permission prompt for an application/test failure.

Document this separately from test failures.

---

# 6. PLAYWRIGHT GLOBAL SETUP — DO IT CORRECTLY

If GoRetailOS needs global auth setup:

Do NOT create a Playwright `test()` inside `global-setup.ts`.

The global setup should be a plain async function such as:

`async function globalSetup()`

exported as default.

If the setup launches Chromium itself, remember:

It does NOT automatically inherit `use.baseURL`.

Therefore either:

* use an absolute URL, or
* explicitly create the context with a base URL.

Example concept:

`PLAYWRIGHT_BASE_URL`

with fallback:

`http://localhost:8080`

Do not call:

`page.goto("/auth")`

inside a raw `chromium.launch()` setup unless a baseURL is configured for that context.

---

# 7. ENVIRONMENT FILE

If Playwright requires `.env.test`, load it explicitly.

If `dotenv` is imported by the Playwright config/setup, ensure it is installed as a dev dependency.

Do not commit real credentials.

Use environment names such as:

* `PLAYWRIGHT_BASE_URL`
* `E2E_OWNER_EMAIL`
* `E2E_OWNER_PASSWORD`
* `E2E_ADMIN_EMAIL`
* `E2E_ADMIN_PASSWORD`
* `E2E_MANAGER_EMAIL`
* `E2E_MANAGER_PASSWORD`
* `E2E_CASHIER_EMAIL`
* `E2E_CASHIER_PASSWORD`
* `E2E_ACCOUNTANT_EMAIL`
* `E2E_ACCOUNTANT_PASSWORD`
* `E2E_INVENTORY_EMAIL`
* `E2E_INVENTORY_PASSWORD`
* `E2E_STAFF_EMAIL`
* `E2E_STAFF_PASSWORD`
* `E2E_CUSTOMER_EMAIL`
* `E2E_CUSTOMER_PASSWORD`

Only require personas genuinely needed by `TEST_CASES.md`.

Never print passwords to test output.

---

# 8. PLAYWRIGHT PROJECT DESIGN

Be very careful with project multiplication.

Do not automatically run every test against every persona.

Classify tests explicitly.

Potential projects may include:

* `visitor`
* `owner`
* `admin`
* `manager`
* `cashier`
* `accountant`
* `inventory`
* `staff`
* `customer`

Do NOT create persona projects that have no meaningful test coverage.

GoRetailOS currently has tenant-based users; confirm whether there is any separate platform-level persona before adding one.

Do not invent a platform-admin project unless the current GoRetailOS code actually contains such a role/surface.

---

# 9. VISITOR VS AUTHENTICATED TEST SCOPING

This is critical.

Tests for:

* invalid login
* unauthenticated redirects
* sign-up
* auth-page validation
* guest storefront

must run in an unauthenticated project/context.

Do not run them with authenticated `storageState`.

Similarly, tests for:

* admin
* POS
* inventory
* purchasing
* accounting
* team
* settings

must not run in the visitor project unless the expected assertion is specifically that access is denied/redirected.

Use explicit project scoping rather than allowing the full test suite to multiply incorrectly across every project.

---

# 10. ROLE-SPECIFIC TEST SCOPING

Do not assume one RBAC test can run unchanged across every user role.

Example:

A test such as:

"Manager must not see X"

must only run against Manager.

A test such as:

"Cashier can open POS"

should run only against Cashier or relevant higher roles.

Use explicit role matrices from current code/migrations.

Do NOT infer role access purely from role names.

---

# 11. LIVE RBAC IS AUTHORITATIVE

A major previous lesson was that documentation can drift.

For GoRetailOS:

Inspect:

* role matrix code
* `permissions`
* `role_permissions`
* relevant migrations
* server guards

If documentation says one thing and live current code says another:

1. determine intended current behaviour
2. update the test expectation to current implementation
3. record documentation drift separately
4. fix documentation if appropriate

Do not mark the application broken solely because an old markdown table is outdated.

But if server enforcement contradicts intended/current role policy, treat that as an actual bug.

---

# 12. QUICK TOUR / FIRST-LOGIN OVERLAYS

GoRetailOS may have a first-login Quick Tour.

Inspect the implementation.

Determine:

* whether it auto-opens
* its localStorage/session flag
* whether it is modal
* whether it uses Radix Dialog or similar
* whether it removes the underlying page from the accessibility tree

If a Quick Tour appears automatically, do NOT make every test dismiss it after navigation.

Prefer adding an init script before navigation that sets the correct "tour completed" flag.

Example concept:

`page.addInitScript(() => localStorage.setItem("...", "..."))`

BUT:

Do not copy the VOXOPlan storage key.

Inspect the GoRetailOS code and use the actual GoRetailOS key/value.

Create separate dedicated tests for the Quick Tour itself.

Normal functional tests should suppress onboarding/tours before page load to avoid flakiness.

---

# 13. DO NOT ASSERT ROUTE NAMES AS PAGE TITLES

Previous tests failed because the route was called "CRM" while the rendered heading was "Customers".

For GoRetailOS:

Before writing assertions, inspect the actual component.

Assert on:

* actual `PageHeader`
* actual heading
* actual accessible button name
* actual labels
* actual dialog title

Do not infer visible text from the route path.

For example:

`/admin/purchasing`

may render:

`Suppliers & Purchase Orders`

or another title.

Inspect first.

---

# 14. SELECTOR STRATEGY

Prefer stable semantic selectors:

1. `getByRole`
2. `getByLabel`
3. `getByPlaceholder`
4. stable `data-testid`
5. stable text

Avoid:

* fragile CSS chains
* nth-child selectors
* internal Tailwind class selectors
* route-name assumptions
* raw email assertions unless actually shown
* selecting by styling

If a critical flow is impossible to automate reliably without a stable selector, add a localized `data-testid` only when justified.

Do not add hundreds of test IDs indiscriminately.

---

# 15. DO NOT MAKE SMOKE TESTS TOO DEEP

A smoke test should prove that a major feature surface works.

It should not necessarily complete an entire complicated workflow.

Examples:

Good smoke test:

* open Products
* open Add Product
* verify key inputs exist
* close

Separate functional test:

* create product
* save
* verify DB
* edit
* sell
* return

Similarly:

Do not turn every smoke test into a full data-creation scenario.

Keep smoke tests:

* fast
* stable
* side-effect-light

Put full workflows in dedicated suites.

---

# 16. TENANT ISOLATION TEST SCOPING

This is extremely important for GoRetailOS.

When testing:

Shop A cannot see Shop B

make sure the test actually runs as a Shop A persona.

Do not run that exact assertion as a Shop B user and then treat legitimate Shop B visibility as a failure.

Use clear fixtures:

* `shopAOwner`
* `shopAAdmin`
* `shopBOwner`

or equivalent.

Use actual record IDs/data from the target tenants.

Do not merely search the page text for "Shop B".

Prefer deterministic test records such as:

`QA-SHOPA-ONLY-PRODUCT`

and:

`QA-SHOPB-ONLY-PRODUCT`

Then verify each tenant's access.

---

# 17. STORAGESTATE RULES

If using role-specific storage states:

Generate them using real login flows.

Store them under a gitignored path such as:

`playwright/.auth/`

or equivalent.

Do not commit secrets/session tokens.

Global setup should fail clearly when required test accounts are missing.

Do not silently continue with unauthenticated state.

---

# 18. SHOP SWITCHING

GoRetailOS allows users to potentially belong to multiple shops.

Create dedicated tests.

Do not accidentally use one multi-shop user for every tenant-isolation case because switching state can create ambiguous expectations.

When testing shop switching:

verify:

* current shop changes
* route remains valid
* data reloads
* old shop data disappears
* local caches do not leak
* branch context updates
* POS register context updates where relevant

---

# 19. TEST DATA MUST BE DETERMINISTIC

Use a prefix such as:

`QA-`

and preferably a unique run ID.

Example:

`QA-${RUN_ID}-Perfume`

Do not depend on random existing production-like records such as:

"Product 1"

or:

"Customer A"

unless the environment is a controlled fixture.

Tests should create or seed their own predictable records whenever possible.

---

# 20. SEPARATE TEST BUGS FROM APPLICATION BUGS

Whenever a test fails, classify the failure before modifying application code.

Use:

### TEST BUG

Examples:

* wrong selector
* wrong role expectation
* wrong project
* modal blocking assertion
* wrong heading
* stale docs
* poor fixture
* incorrect tenant assumption

### APPLICATION BUG

Examples:

* unauthorized mutation succeeds
* stock deducts twice
* incorrect total
* payment proof auto-confirms payment
* cross-shop data leak
* broken button
* incorrect journal
* missing validation

### ENVIRONMENT / INFRASTRUCTURE ISSUE

Examples:

* dev server down
* terminal prompt
* execution policy
* Chromium not installed
* Supabase unavailable
* environment variable missing

Do not "fix" application code when the real issue is the test.

---

# 21. FAILED TEST DEBUGGING PROCESS

For each unexplained failure:

1. inspect Playwright error
2. inspect screenshot
3. inspect trace if available
4. inspect DOM/accessibility tree
5. inspect route source code
6. inspect network/server errors
7. inspect database state if applicable
8. classify the failure
9. fix test or app appropriately
10. rerun only affected test first
11. then rerun surrounding suite

Do not repeatedly run the entire suite while debugging one selector.

---

# 22. MODALS / RADIX DIALOGS

If GoRetailOS uses Radix dialogs:

Remember modal dialogs can mark background content `aria-hidden`.

While a modal is open, underlying headings may not exist in the accessibility tree.

Do not assert page content underneath a modal until the modal is closed.

For tests that intentionally test the modal:

scope locators to the dialog.

Example concept:

`page.getByRole("dialog")`

then search within it.

---

# 23. OFFLINE POS TESTING

GoRetailOS has offline/PWA capabilities.

Do not use generic navigation assumptions.

For Playwright offline tests:

* load/authenticate/setup first
* ensure relevant catalog/cache exists
* then set browser context offline
* perform offline action
* restore connection
* verify reconciliation

Do not attempt initial login while offline unless that exact scenario is being tested.

Document any service-worker/browser constraints you discover.

---

# 24. POS TEST PROJECT SCOPING

POS tests should run only for roles that can actually access the POS.

Check current access from code.

Do not run all POS tests as:

* Accountant
* Inventory
* Staff

unless the expected result is an authorization denial.

Be especially careful with:

* register assignments
* open cash sessions
* inventory state
* parallel execution

Use serial mode or isolated registers where necessary.

---

# 25. TESTS THAT SHOULD NOT RUN IN PARALLEL

Identify shared mutable resources.

Likely examples:

* same register
* same cash session
* same inventory SKU
* same document sequence
* same payment proof
* same supplier invoice
* same offline queue

Either:

* allocate independent fixtures/resources per worker, or
* mark the group serial

Do not hide concurrency issues by disabling all parallelism globally unless necessary.

---

# 26. FULL REGRESSION PROJECT EXPLOSION

Before running the full suite, calculate approximate execution count:

`test cases × Playwright projects`

Avoid accidentally turning 100 logical tests into 800 executions when only 150 are meaningful.

Use:

* project scoping
* role-specific describes
* skip conditions only when appropriate
* dedicated config/testMatch where useful

Prefer tests that naturally select the correct persona rather than excessive runtime skipping.

---

# 27. DOCUMENT SKIPS

A skipped test should have a reason.

Avoid unexplained skips.

Categories may include:

* wrong persona
* external DNS
* manual hardware
* browser limitation
* feature not implemented
* environment-dependent

The final report should distinguish:

* passed
* failed
* skipped by design
* not executed

---

# 28. CUSTOM DOMAIN TESTING

Do not try to alter real DNS as part of routine Playwright execution.

Separate:

### Application-automatable

* add domain UI
* validation
* duplicate domain handling
* primary selection
* route mapping if host can be simulated
* permission controls

from:

### External/manual

* Namecheap DNS
* propagation
* SSL issuance
* public resolver behaviour

Do not mark DNS propagation failure as a GoRetailOS code failure unless the app integration itself is at fault.

---

# 29. PAYMENT PROOF TESTING

GoRetailOS has a sensitive account-transfer flow.

Be extremely strict.

The automated test should distinguish:

1. order created
2. payment proof uploaded
3. proof submitted
4. merchant verifies
5. payment becomes paid

At step 2/3:

**payment must still NOT be considered confirmed.**

Do not write a smoke assertion that merely sees the upload and assumes success.

Also verify role/tenant isolation on proof access.

---

# 30. BACKEND ASSERTIONS

A green UI does not always mean the flow is correct.

For critical workflows, pair Playwright UI actions with backend/database assertions where feasible.

Especially:

* POS sale
* returns
* inventory receiving
* transfer
* stock count
* supplier payment
* expense
* ecommerce checkout
* payment verification
* accounting posting
* tenant isolation

If database assertion utilities are needed, create them carefully and keep tenant credentials/secrets out of browser code.

---

# 31. TEST REPORT / PROBLEM LOG

Create or maintain a GoRetailOS-specific file:

`PLAYWRIGHT_QA_ISSUES.md`

Every meaningful problem during implementation should be logged incrementally.

Use:

## Problem

What failed.

## Type

* Test bug
* Application bug
* Environment issue
* Documentation drift

## Root Cause

Why.

## Resolution

What changed.

## Related Tests

IDs.

## Lesson

How future tests should handle this.

Do NOT copy the previous project's issues into this file as if they happened here.

Only record issues actually encountered in GoRetailOS.

---

# 32. PROACTIVE PRE-FLIGHT CHECK

Before writing the main suites, verify:

* dev server can start
* target URL loads
* Playwright browser installed
* `.env.test` loads
* visitor project works
* owner login works
* storageState works
* Quick Tour suppression works if applicable
* at least one admin route loads
* POS route loads for correct persona
* storefront loads as visitor
* project scoping works

Only after this foundation is stable should you expand to hundreds of tests.

---

# 33. IMPLEMENT IN STAGES

Use stages similar to:

## Stage 0 — Infrastructure

Playwright install/config/env/browser.

## Stage 1 — Authentication

Visitor + login + storage states.

## Stage 2 — P0 Smoke

Admin, POS, storefront.

## Stage 3 — Tenant Isolation + RBAC

## Stage 4 — Catalog + Customers

## Stage 5 — POS + Sales

## Stage 6 — Inventory + Purchasing

## Stage 7 — Accounting + Expenses

## Stage 8 — Storefront + Ecommerce + Payment Proof

## Stage 9 — AI + Data Manager + Domains

## Stage 10 — Responsive + Offline + Accessibility

## Stage 11 — Full Regression

After every stage:

* run that stage
* fix issues
* update `PLAYWRIGHT_QA_ISSUES.md`
* then proceed

Do not wait until the very end to discover infrastructure mistakes.

---

# 34. DO NOT ALTER INTENDED BUSINESS BEHAVIOUR JUST TO MAKE TESTS PASS

Never weaken:

* RBAC
* RLS
* tenant isolation
* payment verification
* inventory validation
* accounting rules
* POS session rules
* overselling restrictions

because a test expected something different.

Correct the test if the expectation was wrong.

Only change application behaviour when the code is actually defective.

---

# 35. REUSE LESSONS, NOT OLD ASSUMPTIONS

The previous project's lessons are patterns.

Do NOT copy:

* old routes
* old localStorage keys
* old role names
* old organization terminology
* old platform-admin assumptions
* old headings
* old test credentials
* old feature expectations

Everything must be mapped to GoRetailOS current code.

---

# 36. FINAL REPORT

When this Playwright stabilization/setup review is complete, report:

* whether Playwright already existed
* dependencies added/changed
* browsers installed
* config changes
* personas/projects created
* Quick Tour handling
* environment setup
* infrastructure issues encountered
* test bugs encountered
* actual application bugs encountered
* issues fixed
* issues deferred
* total tests currently passing/failing/skipped
* path to `TEST_CASES.md`
* path to `PLAYWRIGHT_QA_ISSUES.md`

The ultimate objective is not simply:

**"Playwright runs."**

The objective is:

**A stable, correctly-scoped GoRetailOS E2E testing foundation that another Claude session can continue without repeating known mistakes.**

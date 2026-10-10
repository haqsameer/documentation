You are working inside the **GoRetailOS / Retail OS repository**.

Your task is to perform a **comprehensive code-driven QA audit**, review the implementation, fix safe issues you discover, and produce a **single authoritative Markdown test specification** that another Claude/AI chat can later use to install/configure Playwright and execute the tests.

The main output must be a new file:

`TEST_CASES.md`

This file will become the master QA specification for the application.

Do NOT write Playwright tests in this task unless Playwright already exists and a very small verification is useful.

Another chat will later be instructed to read `TEST_CASES.md`, add/configure Playwright, and automate the scenarios.

The important objective here is therefore:

**Inspect real code → understand real behaviour → identify testable scenarios → fix safe defects → document every relevant test case clearly enough that another AI can automate it without rediscovering the application.**

---

# 1. IMPORTANT WORKING RULES

Before making changes:

1. Read and respect:

   * `AGENTS.md`
   * `README.md`
   * `features.md`
   * `handoff.md`
   * `SCOPE.md`
   * `SCOPE_STATUS.md`
   * `SCOPE_GAPS.md`
   * `QUICKSTART.md`
   * `usage.md`
   * `USER_MANUAL.md`
   * `POS_MANUAL.md`
   * domain/storefront documentation
   * any other current requirement/status/audit documents present in the repository.

2. Inspect the actual source code.

Do NOT assume something works because documentation says it works.

Do NOT assume something is missing because an older audit says it was missing.

The current source code is authoritative.

3. Do not rewrite published Git history.

Do not:

* force push
* rebase published commits
* amend/squash already-pushed commits in a way that rewrites Lovable history

Keep the connected branch in a working state.

4. Avoid destructive database operations.

Do not reset production/shared data.

Do not delete meaningful existing data merely to test something.

5. Do not perform a major architectural rewrite.

If you discover a large architectural issue:

* document it in `TEST_CASES.md` or a review section
* classify its severity
* explain the impact
* do not rebuild a major subsystem unless absolutely necessary to correct a critical defect safely.

---

# 2. FIRST: UNDERSTAND THE APPLICATION FROM CODE

Inspect the repository comprehensively.

Map:

* routes
* layouts
* authentication
* onboarding
* admin portal
* navigation
* POS
* storefront
* customer portal
* server functions
* Supabase/database interaction
* migrations
* RLS
* permissions
* role guards
* storage
* offline/PWA logic
* background/event logic
* accounting
* inventory
* purchasing
* reporting
* AI features
* custom domains
* responsive/mobile behaviour
* error handling
* loading states
* empty states

Do not just inspect route filenames.

Trace important workflows from:

**UI → client function → server function/RPC → database tables → side effects**

For example:

POS Complete Sale

should be traced through:

UI
→ checkout/server function
→ price/tax recalculation
→ order
→ order items
→ payments
→ inventory movements
→ inventory balances
→ accounting journal
→ receipt
→ audit/event records

Do this type of dependency tracing for all important workflows.

---

# 3. PERFORM A CODE REVIEW WHILE BUILDING THE TEST MATRIX

While studying each area, actively look for defects.

Review for:

## Functional defects

* broken buttons
* dead links
* forms that do not submit
* forms that partially save
* incorrect redirects
* missing refresh/invalidation
* stale UI after mutations
* duplicate submissions
* missing validation
* incorrect defaults
* wrong calculations
* wrong status transitions
* unexpected nullable handling
* incorrect loading/error handling
* race conditions
* duplicate transaction risks

## Multi-tenancy

Look extremely carefully for:

* missing `shop_id` filtering
* trusting client-provided shop IDs
* queries that could cross tenants
* incorrect shop switch handling
* tenant-unaware storage paths
* cache leakage between shops

## Authorization

Check BOTH UI and server side.

A hidden menu is not sufficient.

Verify roles cannot directly invoke restricted operations.

Current conceptual roles include:

* Owner
* Admin
* Manager
* Cashier
* Accountant
* Inventory
* Staff

Verify actual code rather than blindly using this list.

## Security

Review:

* RLS
* server authorization guards
* storage permissions
* signed/private URLs
* public storefront data exposure
* cost-price exposure
* service-role usage
* payment-proof privacy
* AI data isolation
* direct server-function access
* unsafe input handling
* raw SQL construction if any
* unsafe bulk operations
* unrestricted IDs
* IDOR-like behaviour

## Financial correctness

Check:

* sale totals
* tax
* discounts
* split payments
* change due
* credit sales
* payment recording
* refund calculation
* partial returns
* accounting journal balance
* AR/AP postings
* supplier payments
* expenses
* payment proof verification
* order payment status
* duplicate payment prevention

## Inventory correctness

Check:

* sale deduction
* return restock
* receiving stock
* transfers
* adjustments
* stock counts
* reserved stock
* overselling rules
* branch/location selection
* offline reconciliation
* duplicate sync

## UX correctness

Look for:

* layout jumps
* flashing/refetching
* button presses with no feedback
* duplicate click vulnerability
* buttons that remain active during requests
* dancing dashboard elements while data loads
* bad modal layering
* rounded modal with square/overflowing overlay/content
* horizontal mobile overflow
* broken drawers
* inaccessible forms
* missing loaders
* missing disabled states
* broken keyboard interactions
* poor responsive behaviour

---

# 4. FIX ISSUES YOU FIND — WITH STRICT LIMITS

You ARE allowed to fix defects while reviewing.

Fix directly when the issue is:

* clearly a bug
* low or medium risk
* localized
* well understood
* does not require a major schema redesign
* does not change intended business behaviour

Examples:

* missing validation
* null crash
* broken link
* wrong route
* missing authorization check
* stale cache invalidation
* bad loading state
* double-click submission issue
* incorrect conditional
* obvious tenant filter omission
* incorrect status handling
* small calculation defect
* responsive overflow
* modal layering issue

For every code fix:

1. understand root cause
2. make the smallest appropriate change
3. verify related callers
4. run relevant type/build/static checks
5. add the corresponding regression test case to `TEST_CASES.md`

Do NOT make speculative changes merely because you prefer a different implementation.

If an issue requires a major change, document:

* problem
* severity
* impact
* recommended future fix
* why you did not change it now

---

# 5. CREATE `TEST_CASES.md`

This is the most important deliverable.

The file must be designed specifically so another Claude chat can later read it and implement Playwright tests.

It must be **execution-oriented**, not a high-level checklist.

Every test case must have a stable ID.

Use IDs such as:

`AUTH-001`

`ONBOARD-001`

`TENANT-001`

`RBAC-001`

`PRODUCT-001`

`POS-001`

`INV-001`

`PURCHASE-001`

`ACCOUNT-001`

`STORE-001`

etc.

Do not renumber existing IDs later unnecessarily.

---

# 6. REQUIRED FORMAT FOR EVERY TEST CASE

Every test case should contain at least:

### ID

Unique stable test ID.

### Area

Functional domain.

### Title

Short human-readable test name.

### Priority

Use:

* P0 — critical business/security/data integrity
* P1 — core functionality
* P2 — important but non-blocking
* P3 — edge/UX/rare scenario

### Test Type

Use one or more:

* Smoke
* Happy Path
* Negative
* Validation
* Permission
* Security
* Tenant Isolation
* Financial Integrity
* Inventory Integrity
* Regression
* Responsive
* Offline
* Accessibility
* Performance
* Concurrency
* Integration
* Data Integrity

### Suitable for Playwright?

Specify:

* Yes — normal E2E
* Yes — requires API/database setup
* Partial — needs server/database assertion
* Manual
* Not suitable for browser automation

### Roles

Which user role(s) are required.

### Preconditions

Exact state needed before starting.

Example:

* Shop exists
* Owner authenticated
* Branch A exists
* Register A exists
* Variant has 10 units at Location A
* Cash session is open

### Test Data

Give concrete example values wherever practical.

Example:

Product:

* Name: `QA Perfume 100ml`
* SKU: `QA-PERF-100`
* Price: `100`
* Cost: `60`
* Tax: `0%`
* Opening stock: `10`

Prefer a consistent `QA-` naming convention so automation can find and clean its own records.

### Route

Exact route if known.

Example:

`/admin/products`

### Selectors / UI Targets

Do NOT invent selectors that do not exist.

Instead record the actual visible labels, buttons, placeholders, headings, accessible names, or stable existing attributes found in code.

For example:

* button: `Add Product`
* field label: `Product Name`
* button: `Save`
* table text: product SKU

If stable `data-testid` attributes already exist, document them.

If automation would be very brittle without a stable selector, note:

`Recommended test id: data-testid="..."`

Do NOT unnecessarily modify the whole app to add test IDs in this task.

### Steps

Numbered, explicit browser actions.

### Expected UI Result

What the user should see.

### Expected Backend / Data Result

What must have changed beyond the visible UI.

This is EXTREMELY IMPORTANT.

For example after a sale:

* order exists
* correct order channel
* correct line quantity
* correct payment
* inventory deducted once
* inventory movement created
* balanced journal entry created
* receipt number assigned
* event/audit record created if applicable

### Side Effects

Anything else expected to change.

### Negative Assertions

State things that MUST NOT happen.

Example:

* inventory must not be deducted twice
* cashier must not see cost price
* another shop must not see the order
* screenshot upload must not mark order paid

### Cleanup

How automation should clean/reset the test state if necessary.

### Notes for Playwright

Anything the later automation chat must know.

---

# 7. CREATE A TEST DATA STRATEGY

Near the beginning of `TEST_CASES.md`, define reusable test entities.

For example:

## QA Shop A

Primary shop for normal tests.

## QA Shop B

Used for tenant-isolation tests.

## Users

Create or identify test users for:

* Owner
* Admin
* Manager
* Cashier
* Accountant
* Inventory
* Staff
* Customer
* Guest

Do NOT expose secrets in the Markdown file.

Instead define expected environment variable names such as:

`E2E_OWNER_EMAIL`

`E2E_OWNER_PASSWORD`

`E2E_ADMIN_EMAIL`

`E2E_ADMIN_PASSWORD`

etc.

If multiple shops are required, define them consistently.

---

# 8. AUTHENTICATION TEST CASES

Cover everything you can determine exists, including:

* sign up
* sign in
* sign out
* invalid password
* unknown account
* validation
* session persistence
* unauthenticated protected-route access
* authenticated user accessing `/auth`
* Google login UI where practical
* password-related flows if implemented
* user with multiple shops
* shop switching
* expired/invalid session behaviour
* direct navigation to protected URLs

---

# 9. ONBOARDING TEST CASES

Cover:

* first-time user onboarding
* required fields
* invalid values
* shop creation
* branch provisioning
* register provisioning
* inventory location provisioning
* default chart of accounts
* payment methods
* tax defaults
* document sequences
* storefront defaults
* owner membership
* retry/double submission
* navigating back
* duplicate shop creation protection if relevant

After onboarding, assert all required dependent records exist.

---

# 10. MULTI-TENANCY TESTS — VERY HIGH PRIORITY

Create extensive tenant isolation cases.

Use:

**Shop A**
and
**Shop B**

Verify Shop A cannot access Shop B data through:

* UI
* URL parameters
* IDs
* server calls
* query parameters
* storage
* reports
* AI
* products
* variants
* orders
* customers
* suppliers
* purchases
* inventory
* accounting
* audit logs
* payment proofs
* storefront administration
* settings
* data manager
* custom domains

Test shop switching.

Verify state/cache does not leak after switching shops.

These tests are P0.

---

# 11. RBAC / PERMISSION MATRIX

Create explicit tests for every relevant role.

Do not only check navigation.

Test:

1. visibility
2. direct URL access
3. direct action/API/server-function authorization where reasonably testable

Cover:

* Owner
* Admin
* Manager
* Cashier
* Accountant
* Inventory
* Staff

Create a matrix in the file mapping routes/actions by role.

Then create detailed cases for high-risk actions.

Include things such as:

* POS access
* sales access
* returns
* payment recording
* register operations
* catalog modification
* inventory modification
* purchasing
* accounting
* storefront configuration
* AI
* team management
* audit
* settings
* data manager
* custom domain management

---

# 12. DASHBOARD

Test:

* page loads
* KPIs render
* empty state
* populated state
* branch switch
* shop switch
* loading behaviour
* no severe layout shifting
* no buttons/cards jumping while loading
* trend data
* recent activity if present
* responsive layout
* dark/light modes if applicable
* navigation cards/links

---

# 13. PRODUCT / CATALOG TESTS

Cover:

* create product
* edit product
* archive/deactivate
* publish/unpublish
* Available for Sale toggle
* category
* nested category if supported
* brand
* tax rate
* product images
* SKU
* duplicate SKU
* barcode
* duplicate barcode
* EAN validation
* auto/generated barcodes if implemented
* variant products
* attribute creation
* multi-variant matrix
* variant pricing
* variant stock
* compare-at price
* cost privacy
* price changes
* price history
* product search
* filtering
* server-side pagination if present
* CSV import
* XLSX import
* invalid import
* mapping
* preview
* labels
* barcode printing UI
* internal item hidden from POS
* internal item hidden from storefront
* internal item remains usable in purchasing/inventory

Include edge cases such as:

* zero price
* negative price
* extremely long names
* special characters
* duplicate combinations
* no variants
* inactive category
* inactive brand

---

# 14. POS — VERY DETAILED

POS is a critical area.

Create exhaustive cases for:

## Register/session

* no register
* assigned register
* unauthorized register
* open session
* duplicate open session
* opening float
* close session
* counted cash
* variance
* cash in
* cash out

## Product discovery

* scan barcode
* enter SKU
* fuzzy search
* category filter
* keyboard navigation
* stock badges
* unavailable-for-sale product hidden
* inactive product hidden

## Cart

* add item
* repeated add
* increase quantity
* decrease quantity
* remove line
* line discount
* order discount
* attach customer
* detach/change customer
* hold sale
* resume held sale
* cancel cart

## Inventory restrictions

* sell within stock
* exact remaining stock
* attempt oversell
* permitted role oversell
* prohibited role oversell
* non-stock item if supported

## Payment

* cash
* supported non-cash methods
* split payments
* under-allocation
* exact allocation
* overpayment/change
* credit/udhaar if implemented
* credit limit
* duplicate click on Complete Sale
* network interruption during submit
* server recalculation of price/tax
* stale frontend price

## Completion assertions

Every completed sale should verify:

* single order
* line items
* price
* discount
* tax
* total
* payment rows
* payment status
* inventory deduction
* inventory movement
* accounting posting
* customer metrics where appropriate
* receipt
* document sequence
* audit/event logging where applicable

## Receipt

* 58mm
* 80mm if configurable
* print
* reprint
* correct totals
* correct shop details
* document number

## Customer display

If implemented:

* opens
* updates cart
* updates quantity
* updates total
* sale complete/reset

## Keyboard shortcuts

Test:

* F2
* F4
* F6
* F8
* F9
* Esc

where browser automation can reasonably do so.

---

# 15. OFFLINE POS

Create a dedicated section.

Test:

* app offline indicator
* catalog available offline
* barcode cache
* offline sale queue
* reconnect
* sync
* no duplicate order
* duplicate `local_uuid` handling
* multiple pending sales
* sync failure
* retry
* conflict generation
* conflict resolution UI
* stale pricing version
* stale tax version
* insufficient stock during reconciliation
* multiple devices if testable
* monotonic local sequence if relevant
* browser reload while sale pending
* IndexedDB persistence

Clearly label tests that need browser context manipulation such as:

`context.setOffline(true)`

for future Playwright implementation.

---

# 16. RETURNS / REFUNDS / EXCHANGES

Cover:

* full return
* partial return
* single item from multi-line order
* partial quantity
* return exact sold quantity
* attempt over-return
* second return after partial return
* invalid order
* refund
* restock
* non-restock condition if supported
* reason
* notes
* authorization
* inventory movement
* inventory balance
* accounting reversal/refund posting
* payment/refund status
* original sale integrity

If exchange is implemented as return + new sale, document that exact expected model.

---

# 17. CUSTOMERS / CRM

Cover:

* create customer
* edit
* name
* phone
* email
* city
* tags
* notes
* customer group
* search
* filtering
* purchase history
* total spend
* AOV
* last purchase
* segmentation
* repeat customer
* guest order
* customer-linked POS sale
* customer-linked online order
* duplicate email/phone behaviour
* cross-tenant customer isolation

---

# 18. INVENTORY

Cover:

* opening stock
* inventory balance
* stock movement
* manual adjustment increase
* adjustment decrease
* permission controls
* transfers
* create transfer
* send/receive lifecycle
* partial/invalid receipt if supported
* same-location protection
* stock counts
* stock discrepancy
* count posting
* reorder point
* low-stock alert
* reserved stock
* negative stock controls
* branch/location switching
* inventory reports
* CSV export
* date filters
* inventory value

Verify stock movements and balances remain mathematically consistent.

---

# 19. PURCHASING

Cover:

* supplier create/edit
* PO create
* multiple lines
* quantities
* cost
* PO edit before receiving
* cancel
* partial receive
* full receive
* over-receive prevention
* goods receipt
* stock update
* average cost update
* supplier invoice
* supplier payment
* outstanding balance
* accounting posting
* purchase reports
* date filters
* export
* permissions
* cross-tenant isolation

Verify receiving inventory happens exactly once.

---

# 20. SALES / ORDERS

Cover:

* POS orders
* ecommerce orders
* order list
* filters
* search
* order details
* payment status
* fulfillment status
* delivery status if implemented
* record payment
* partial payment if supported
* credit balance
* duplicate payment protection
* returns
* printing/exporting where available
* role restrictions

---

# 21. ACCOUNTING / FINANCE

Create detailed financial integrity tests.

Cover:

* chart of accounts
* journal entries
* journal balance
* sale posting
* sale return posting
* purchase posting
* supplier payment
* expense
* customer payment
* credit sale
* accounts receivable
* accounts payable
* P&L
* balance sheet
* cash flow
* trial balance
* general ledger
* ageing reports
* date filters
* branch filters if supported
* CSV/print
* duplicate posting prevention

Critical invariant:

**Debits must equal credits.**

Document expected account movements whenever the code makes them clear.

---

# 22. EXPENSES

Cover:

* create
* validation
* category
* account
* amount
* date
* attachment
* approval if implemented
* rejection
* recurring configuration if implemented
* accounting posting
* reporting
* permissions
* storage security

---

# 23. STOREFRONT ADMIN

Cover all current configurable storefront capabilities found in code.

Examples may include:

* branding
* logo
* colours
* themes
* banners
* sections
* pages
* navigation
* policies
* translations
* currency
* configuration
* publish state
* domain management

Do not create tests for a feature merely because it existed in an older template.

Verify current code.

---

# 24. PUBLIC STOREFRONT

Test both:

`/store/:shopSlug/*`

and custom-domain clean routing where the current code supports it.

Cover:

* homepage
* shop/catalog
* categories
* product detail
* product variants
* pricing
* compare-at price
* stock
* unavailable product
* search
* filters
* category filter
* cart
* cart quantity
* remove
* wishlist
* compare
* guest checkout
* customer checkout
* account
* addresses
* order tracking
* CMS pages
* policies
* responsive navigation
* empty states
* invalid product slug
* invalid shop slug
* inactive shop if applicable

Verify cost price is NEVER exposed publicly.

---

# 25. ECOMMERCE CHECKOUT

Only test payment methods actually supported by code.

Current intended modes are:

* Cash on Delivery
* Account / Bank Transfer

Do NOT assume Stripe/PayPal/card gateways exist.

Cover:

## COD

* successful order
* unpaid/pending payment state as designed
* stock
* accounting behaviour
* fulfillment status

## Account transfer

Test very carefully:

* bank details displayed
* order created
* order initially unpaid
* AR if applicable
* upload payment screenshot
* private storage
* proof-submitted state
* screenshot upload DOES NOT mark paid
* merchant preview
* merchant verify
* verification marks payment correctly
* journal/payment created once
* rejection
* rejection reason
* customer sees rejection
* re-upload after rejection
* duplicate verification protection
* unauthorized user cannot view proof
* another tenant cannot view proof

This is P0.

---

# 26. CUSTOM DOMAINS

Inspect current implementation and create cases for:

* add domain
* domain normalization
* duplicate domain
* verification state
* invalid domain
* ownership verification flow
* mark verified behaviour
* make primary
* multiple domains
* primary redirect
* clean root storefront route
* clean `/product/:slug`
* `/cart`
* `/checkout`
* legacy `/store/:shopSlug/*`
* unknown hostname
* unverified hostname
* cross-shop domain isolation

Do not attempt real DNS changes as part of automated tests unless an isolated test environment exists.

Separate pure application tests from external DNS/manual verification tests.

---

# 27. AI BUSINESS ASSISTANT

Inspect actual tools and permissions.

Test:

* normal question
* business-data question
* tenant scope
* unauthorized role
* action proposal
* approval required
* approve
* reject
* execute
* execution result
* invalid action
* bad IDs
* lookup resolution
* auditability
* cross-tenant isolation
* knowledge base
* business memory
* alerts if implemented

For write actions such as:

* create PO
* adjust stock
* record expense
* record payment

verify the assistant cannot bypass normal business rules.

---

# 28. STOREFRONT AI SALESMAN

If currently implemented, test:

* guest access
* product search
* price filter
* category filter
* on-sale filter
* recommendations
* stock correctness
* pricing correctness
* cart proposal/actions
* invalid product
* hidden/internal products excluded
* cost information inaccessible
* tenant isolation
* customer-specific context only when authorized

---

# 29. STORE DATA MANAGER

This is high-risk.

Create detailed P0/P1 cases for:

* owner access
* admin access
* unauthorized roles
* allowed tables
* protected tables
* filters
* AND
* OR
* operators
* preview count
* bulk update
* update validation
* bulk delete
* typed DELETE confirmation
* row limit
* inserts
* JSON import
* CSV import
* `shop_id` stamping
* attempt to insert another shop ID
* reset modes
* typed RESET confirmation
* foreign-key dependency resolver
* set-null
* delete-children
* audit log
* no raw SQL exposure
* cross-tenant attack attempts

Use isolated QA data only.

---

# 30. REPORTS

For every report family found in code:

* sales
* purchases
* inventory
* accounting
* customers
* price history
* cost history
* other current reports

Test:

* default load
* date range
* invalid date range
* empty period
* populated period
* totals
* pagination
* filter
* print
* CSV
* tenant scope
* branch scope
* role access

Where possible, calculate expected totals from controlled QA transactions rather than merely checking that a number exists.

---

# 31. SEARCH / FILTER / PAGINATION

For all major data tables, test:

* search
* case sensitivity where relevant
* partial matching
* no results
* filters
* multiple filters
* clear filters
* sort
* pagination
* page size if supported
* state after mutation
* large datasets

Identify which tables are server-side and which are client-side from code.

Document that in test notes.

---

# 32. RESPONSIVE / MOBILE UI

Test representative viewports such as:

* 375×812
* 390×844
* 768×1024
* 1280×720
* 1440×900

Check:

* no horizontal overflow
* admin drawer
* collapsed sidebar
* tables → mobile cards
* modals fit screen
* dropdowns remain visible
* buttons accessible
* POS mobile cart
* POS payment sheet
* storefront navigation
* checkout
* customer account
* forms
* long text
* empty states

Pay special attention to modal border/overlay/layer clipping issues.

---

# 33. LOADING / UI STABILITY

Create specific tests for perceived-quality issues.

Check that:

* buttons show submitting state
* buttons disable where duplicate submission would be dangerous
* loaders/skeletons do not cause severe layout shift
* dashboard cards do not jump/dance
* filters do not cause unnecessary full-page flashes
* mutation does not unnecessarily remount the page
* modals don't flash
* data refresh doesn't destroy user scroll unnecessarily
* errors are visible
* successful operations give feedback

These should be explicit regression tests.

---

# 34. ERROR HANDLING

Create tests for:

* failed server request
* unauthorized
* forbidden
* not found
* invalid data
* timeout-like failures where practical
* disconnected internet
* storage upload failure
* failed payment-proof upload
* failed checkout
* failed POS sync
* failed report load

Application should not silently fail.

---

# 35. CONCURRENCY / DUPLICATION

Identify high-risk transactional actions and create test cases for:

* double-click Complete Sale
* two sale requests
* duplicate sync
* duplicate payment verification
* duplicate customer payment
* duplicate stock receipt
* simultaneous register session open
* simultaneous document number allocation
* simultaneous stock deduction
* concurrent inventory update
* repeated return submission
* repeated expense submission

Some may require API-level or parallel Playwright requests.

Label them appropriately.

---

# 36. SECURITY / ABUSE CASES

Create test cases for:

* manipulating IDs in URLs
* another tenant's record ID
* another tenant's storage path
* direct protected function call
* unauthorized mutation
* public cost-price leakage
* payment-proof leakage
* AI cross-tenant leakage
* admin-only pages
* XSS-like text in product/customer fields
* malicious filenames
* invalid MIME upload
* dangerous import values
* bulk manager attempts
* storefront hidden product access
* inactive product direct URL access

Do not perform destructive attacks against shared production infrastructure.

---

# 37. ACCESSIBILITY

Add practical tests for:

* form labels
* button names
* keyboard navigation
* focus
* modal focus behaviour
* Escape
* visible focus
* accessible names
* required fields
* validation messages
* colour-only status communication

These can later be combined with Playwright/axe if the automation chat chooses.

---

# 38. PERFORMANCE-SENSITIVE CASES

Do not benchmark deeply here.

But document scenarios that should be checked with meaningful datasets:

* 1,000+ products
* large product search
* large customer list
* large sales list
* large inventory list
* report date range
* POS product lookup
* shop switching
* dashboard load

Identify obvious N+1 or client-side heavy behaviour during the code review.

Fix only if safe and localized.

---

# 39. CREATE A PLAYWRIGHT AUTOMATION PLAN INSIDE THE FILE

At the end of `TEST_CASES.md`, add:

# Playwright Automation Guidance

Include recommendations for the future automation chat.

Document:

## Suggested project structure

Example:

`tests/e2e/auth/`

`tests/e2e/onboarding/`

`tests/e2e/pos/`

`tests/e2e/inventory/`

`tests/e2e/purchasing/`

`tests/e2e/storefront/`

`tests/e2e/security/`

`tests/e2e/tenant-isolation/`

## Fixtures

Recommend fixtures for:

* authentication
* Owner
* Admin
* Manager
* Cashier
* Accountant
* Inventory
* Staff
* Customer
* Shop A
* Shop B
* seeded product
* register
* inventory
* supplier

## Storage state

Recommend role-specific Playwright storage states instead of repeatedly signing in where safe.

## Data isolation

Every automated run should create/use records prefixed with:

`QA-`

or a run-specific identifier.

Example:

`QA-${RUN_ID}-Product`

## Cleanup

Explain safe cleanup strategy.

Do not reset the entire database.

## Test dependencies

Prefer independent tests.

Do not create large chains such as:

Test 1 creates product
→ Test 2 assumes product exists
→ Test 3 assumes sale exists

Instead use fixtures/setup or grouped serial tests only where the flow genuinely requires state progression.

## Database assertions

Some scenarios need backend verification.

Mark those tests so the Playwright implementation chat knows where browser-only assertions are insufficient.

## Screenshots

Recommend screenshots automatically on failure.

## Trace/video

Recommend Playwright trace on first retry.

## Parallelization

Identify test groups that should NOT run concurrently because they manipulate shared:

* register sessions
* inventory
* document sequences
* offline sync
* payment proofs

---

# 40. CREATE A PLAYWRIGHT EXECUTION MATRIX

Add a final table such as:

| Suite      | IDs      | Browser           | Role            | DB Setup                | Parallel?  | Priority |
| ---------- | -------- | ----------------- | --------------- | ----------------------- | ---------- | -------- |
| Auth       | AUTH-*   | Chromium          | mixed           | low                     | yes        | P0/P1    |
| Tenant     | TENANT-* | Chromium          | Owner/Admin     | Shop A+B                | limited    | P0       |
| POS        | POS-*    | Chromium          | Cashier/Manager | seeded register/product | no/limited | P0       |
| Storefront | STORE-*  | Chromium + Mobile | Guest/Customer  | seeded catalog          | yes        | P1       |

Build this from the actual test cases.

---

# 41. TEST CASE COVERAGE SUMMARY

At the end, give counts:

* Total test cases
* P0
* P1
* P2
* P3
* Playwright automatable
* Needs DB/API assertion
* Manual/external
* Security
* Tenant isolation
* Responsive
* Offline

Also show counts per functional area.

This will help the next chat split automation into batches.

---

# 42. CODE REVIEW FINDINGS

Add a section near the end:

# Code Review Findings

Separate:

## Fixed During This Review

For every issue:

* ID
* severity
* affected file(s)
* root cause
* change made
* regression test ID

Example:

`FIX-001`

Severity: P1

Problem:
Complete Sale remained clickable during submission.

Risk:
Duplicate sale request.

Fix:
Disabled action while request is active.

Regression:
`POS-042`

## Open Findings

For issues not changed:

* ID
* severity
* description
* affected files
* impact
* why not fixed
* recommended approach
* related test IDs

Do not hide problems merely because you did not fix them.

---

# 43. CHECK DOCUMENTATION AGAINST CODE

While working, identify documentation drift.

If the docs claim something that current source code does not support, record it.

Do not silently trust:

* feature register
* handoff
* user manual
* POS manual
* scope/status docs

Where easy and appropriate, correct clearly stale technical documentation.

Do not rewrite entire documentation files unnecessarily.

---

# 44. IMPORTANT BUSINESS RULES THAT MUST BE PRESERVED

Do not accidentally introduce unsupported features while testing/fixing.

In particular:

* Do not add Stripe.
* Do not add PayPal.
* Do not add online card processing.
* Do not introduce an unnecessary vendor-product master association.
* Do not build marketplace functionality.
* Do not build loyalty simply because tests reveal it is absent.
* Do not start speculative Phase 2 work.

The current ecommerce payment model should be validated against actual code, with the intended model being:

* Cash on Delivery
* Account/Bank Transfer
* payment-proof upload
* merchant verification
* screenshot upload alone must NOT equal payment confirmation

---

# 45. TEST THE ACTUAL IMPLEMENTATION, NOT ONLY THE HAPPY PATH

For every core feature, think in this pattern:

1. Does it work normally?
2. What if required data is missing?
3. What if input is invalid?
4. What if the user lacks permission?
5. What if the record belongs to another shop?
6. What if the request is submitted twice?
7. What if another action occurs concurrently?
8. What if the network fails?
9. What if the browser refreshes?
10. What downstream records should change?
11. What records must NOT change?
12. What happens on mobile?
13. What happens with an empty database?
14. What happens with a large dataset?
15. What happens if the user manually edits the URL/ID?

Use this reasoning throughout the test specification.

---

# 46. VERIFY THE PROJECT AFTER FIXES

After completing safe fixes:

Run the repository's appropriate checks.

At minimum, inspect `package.json` first and then run applicable commands such as:

* typecheck
* lint
* build
* existing tests

Do not invent commands that do not exist.

If `npm run build` is the appropriate build command, run it.

Fix new errors introduced by your changes.

Do not get distracted into repairing unrelated large pre-existing issues unless they block meaningful verification.

Document any remaining build/test issues.

---

# 47. FINAL DELIVERABLES

When finished, I expect:

1. `TEST_CASES.md`

   * exhaustive
   * code-grounded
   * execution-oriented
   * suitable as direct input to a future Playwright automation chat

2. Safe code fixes for genuine defects discovered during inspection.

3. Any small documentation corrections that were clearly required.

4. A concise final response telling me:

   * total number of test cases created
   * areas covered
   * P0/P1 count
   * number of issues found
   * number fixed
   * number intentionally deferred
   * build/typecheck/lint/test status
   * exact path of `TEST_CASES.md`

---

# 48. DO NOT STOP AT A SMALL TEST LIST

This is not a request for 30–50 generic test cases.

The application is a broad Retail Operating System.

I expect you to inspect the code comprehensively and create **as many meaningful test cases as the implemented functionality requires**.

Hundreds of cases are acceptable if justified.

Do not inflate the count with meaningless duplicates, but do not collapse materially different scenarios into vague statements such as:

“Test POS works.”

A future automation agent should be able to turn individual cases into individual Playwright tests or clearly grouped test blocks.

The quality bar is:

**Another AI should be able to read `TEST_CASES.md` and begin Playwright implementation immediately without having to rediscover how the application works.**

# Documentation Screenshot Review

Review of the screenshots captured from the live GoRetailOS application and inserted into the
user-facing documentation. This document records what was captured, where each image is used,
and any notes or drift found during the review.

## Summary

- **Source:** Live production site `https://goretailos.com` (shop: Trendies Store)
- **Capture tooling:** `tests/e2e/docs-screenshots.spec.ts` + `playwright.screenshots.config.ts`
- **Viewport:** 1440 × 900
- **Total screenshots captured:** 18
- **Total image references in docs:** 39 (across 3 documents)
- **Broken references:** 0
- **Missing files:** 0

## Screenshot inventory

| # | Screenshot | File exists | Referenced in | Notes |
| --- | --- | --- | --- | --- |
| 1 | Staff sign-in | ✅ | USER_MANUAL, usage | Auth screen |
| 2 | Admin dashboard | ✅ | USER_MANUAL, usage, POS_MANUAL | KPI dashboard |
| 3 | Products list | ✅ | USER_MANUAL, usage | Catalog |
| 4 | Stock movements | ✅ | USER_MANUAL, usage | Inventory |
| 5 | Purchase orders | ✅ | USER_MANUAL, usage | Purchasing |
| 6 | Accounting ledger | ✅ | USER_MANUAL, usage | Accounting |
| 7 | Customers list | ✅ | USER_MANUAL, usage | Customer directory |
| 8 | Shop settings | ✅ | USER_MANUAL, usage | Settings |
| 9 | Team members | ✅ | USER_MANUAL, usage | Team & roles |
| 10 | AI business assistant | ✅ | USER_MANUAL, usage | AI |
| 11 | Data manager | ✅ | USER_MANUAL, usage | Bulk data tools |
| 12 | POS terminal | ✅ | USER_MANUAL, usage, POS_MANUAL | POS |
| 13 | Storefront home | ✅ | USER_MANUAL, usage | Storefront |
| 14 | Storefront shop | ✅ | USER_MANUAL, usage | Product grid |
| 15 | Product detail | ✅ | USER_MANUAL, usage | Product page |
| 16 | Cart | ✅ | USER_MANUAL, usage | Cart |
| 17 | Checkout | ✅ | USER_MANUAL, usage | Checkout |
| 18 | Track order | ✅ | USER_MANUAL, usage | Order tracking |

## Reference verification

All 39 image references across `USER_MANUAL.md`, `usage.md` and `POS_MANUAL.md` were checked
against the files on disk. Every referenced path resolves to an existing PNG file. No broken
links, no case mismatches, no missing images.

### Per-document reference counts

| Document | References | Unique images | Broken |
| --- | --- | --- | --- |
| `USER_MANUAL.md` | 18 | 18 | 0 |
| `usage.md` | 18 | 18 | 0 |
| `POS_MANUAL.md` | 3 | 2 | 0 |

## Quality review

Each screenshot was visually inspected for:

- **Blank / partial loads** — none found; all pages rendered fully.
- **Sensitive data** — screenshots contain real demo data from the owner's shop (Trendies Store):
  product names, prices in PKR, customer names/emails, and order references. This is the owner's
  own production data and is appropriate for the documentation. No payment card numbers, CVVs,
  or credentials are visible.
- **Layout** — all screenshots captured at 1440 × 900 desktop viewport; responsive mobile layouts
  are not shown (documentation targets desktop readers).

## Notes / drift

- **Storefront capture retry:** The first run of the screenshot script failed on the storefront
  pages with a transient `net::ERR_CONNECTION_RESET`. Re-running the storefront tests succeeded —
  this was a network hiccup, not an application defect.
- **Re-capture policy:** Re-run `npx playwright test --config=playwright.screenshots.config.ts`
  after any significant UI change to keep the documentation accurate.
- **Manifest:** `docs/screenshots/README.md` lists every image with its route, usage and description.

## Conclusion

The documentation screenshot phase is **complete**. All 18 screenshots are captured from the live
app, reviewed for quality, and inserted into the three target documents. All image references are
valid and resolve to existing files.
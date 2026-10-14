# QA Report — Site Footer

> Spec: spec.md
> Date: 2026-10-14 · QA by: Ori Chai Matan · Mode: setup
> Verdict: PASS WITH NOTES

## Acceptance Criteria

| AC | Criterion | Result | Evidence |
|---|---|---|---|
| AC1 | Footer on every page from the Strapi `footer` single type | pass | **Runtime:** one `footer` on Home, `/products`, a PDP and `/account` (signed in); columns `Shop`, `Account`; tagline and copyright from Strapi. **Test:** `SiteFooter.test.tsx` (brand, columns in order, bottom bar) |
| AC2 | `GET /api/footer` public and read-only | pass | **Runtime:** public GET → 200; public `PUT /api/footer` → 403. **Test:** `cms/tests/footer/bootstrap.test.ts` (public `find` only) |
| AC3 | Create-only `npm run seed:footer` with only working links | pass | **Runtime:** 404 before → "created the footer with 2 columns" → rerun "skipped"; content matches (Shop: Surfboards / Wetsuits / Apparel; Account: Sign In / Create Account / Wishlist / Order History; no legal links). **Test:** `seed-footer.test.ts` (create-only, seed data, unsafe URLs rejected, production guard) |
| AC4 | Internal links via `next/link`, http(s) anchors, unsafe hrefs dropped | pass | **Runtime:** clicking "Wetsuits" navigates to `/products/category/wetsuits`. **Tests:** `footer.test.ts` (`javascript:`, `//host`, `data:` dropped; empty columns dropped; `isSafeHref`), `SiteFooter.test.tsx` (hrefs) |
| AC5 | Fallback bar when the footer is missing or Strapi is down | pass | **Tests:** `getFooter` returns null on 404 / 500 / network / bad body; `SiteFooter` renders "WESTLINE" + the default copyright with no links. Not exercised live (see Notes) |
| AC6 | Layout at 1280/760/375 and edits within 60s | pass | **Runtime:** 4-column grid at 1280; `gridTemplateColumns` 2 columns at 760 and 375; no horizontal overflow. **Test:** `getFooter` requests with `revalidate: 60` |
| AC7 | My Account has no footer of its own | pass | **Runtime:** `/account` has exactly 1 `footer`, with a 64px gap above it. **Test:** `AccountPage.test.tsx` (no own `contentinfo`) |

## Checks Run

- frontend: `npx vitest run` — pass (99 files, 890 tests); `npx tsc --noEmit` — pass; `npm run lint` — pass (pre-existing font warning); `npm run build` — pass (dev server stopped)
- cms: `npx tsc --noEmit` — pass; `npm test` — pass (22 files, 310 tests)

## Issues Found

(none)

## Notes

- **Not exercised live:** the AC5 fallback (deleting the seeded footer document) and the 60s admin edit in AC6. Both are covered by unit tests and by the same `strapiFetch` path the hero uses.
- **Found and fixed during the live check:** My Account's last section touched the footer (its old bar had a top margin). The page now has `pb-16`.
- **Pre-existing test stderr:** `SiteNav.test.tsx`, `CartDrawer.test.tsx`.

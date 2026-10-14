# Tasks — Site Footer

> Epic: storefront-catalog
> Spec: 2026-10-14-site-footer
> Branch: feat/storefront-catalog/site-footer

## Backend (cms)

- [x] T1 Failing tests: `cms/tests/footer/seed-footer.test.ts` (create-only, payload shape, production guard, seed data) and `cms/tests/footer/bootstrap.test.ts` (public `find` granted) (AC2, AC3)
- [x] T2 Add `shared.link` + `shared.footer-column` components, the `api::footer.footer` single type (core router/controller/service), `bootstrap/footer.ts` registered in `src/index.ts`, `scripts/seed-footer.js` + `seed/footer/footer.json` + `npm run seed:footer` to pass T1
- [x] T3 Live: `GET /api/footer` 404 before the seed; `npm run seed:footer` → 200 with columns; rerun → skipped (AC2, AC3) **Verified 2026-10-14**: 404 before the seed; `npm run seed:footer` created 2 columns; rerun skipped; public `PUT /api/footer` → 403.

## Frontend

- [x] T4 Failing tests: `getFooter` in `__tests__/lib/strapi/footer.test.ts` (normalise, drop unsafe hrefs and empty columns, null on 404 / error) (AC4, AC5)
- [x] T5 Implement `lib/strapi/footer.ts`
- [x] T6 Failing tests: `SiteFooter` (brand, columns in order, bottom bar, next/link vs anchor, fallback bar) (AC1, AC4, AC5)
- [x] T7 Implement `components/site-footer/` (component, texts, styles, barrel)
- [x] T8 Render `<SiteFooter />` in the locale layout; remove My Account's footer bar and its strings/styles; update affected tests; Changelog rows on account-signup-login (layout, index.ts) and account-profile (AC14 superseded) (AC1, AC7)

## Verification

- [x] T9 Full verify (dev server stopped for the build): frontend vitest/tsc/lint/build, cms tsc/test **2026-10-14:** frontend 99 files / 890 tests, tsc, lint, build pass; cms 22 files / 310 tests.
- [x] T10 Live browser check: footer on Home, catalog, a PDP and My Account (one footer); links navigate; 1280/760/375; fallback bar with the footer document missing; an admin edit shows within 60s (AC1, AC5, AC6) **Done 2026-10-14** for Home, catalog, PDP, My Account (one footer each; account gap fixed with `pb-16`), link navigation, 760/375 (2 columns, no overflow). The fallback bar and the 60s admin edit are covered by tests, not exercised live — see qa-report.md.
- [x] T11 QA against AC1-AC7 (qa skill) → qa-report.md **Run 2026-10-14:** PASS WITH NOTES, 7/7 AC — see qa-report.md.

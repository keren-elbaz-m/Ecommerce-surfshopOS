# Tasks — Site Footer

> Epic: storefront-catalog
> Spec: 2026-10-14-site-footer
> Branch: feat/storefront-catalog/site-footer

## Backend (cms)

- [ ] T1 Failing tests: `cms/tests/footer/seed-footer.test.ts` (create-only, payload shape, production guard, seed data) and `cms/tests/footer/bootstrap.test.ts` (public `find` granted) (AC2, AC3)
- [ ] T2 Add `shared.link` + `shared.footer-column` components, the `api::footer.footer` single type (core router/controller/service), `bootstrap/footer.ts` registered in `src/index.ts`, `scripts/seed-footer.js` + `seed/footer/footer.json` + `npm run seed:footer` to pass T1
- [ ] T3 Live: `GET /api/footer` 404 before the seed; `npm run seed:footer` → 200 with columns; rerun → skipped (AC2, AC3)

## Frontend

- [ ] T4 Failing tests: `getFooter` in `__tests__/lib/strapi/footer.test.ts` (normalise, drop unsafe hrefs and empty columns, null on 404 / error) (AC4, AC5)
- [ ] T5 Implement `lib/strapi/footer.ts`
- [ ] T6 Failing tests: `SiteFooter` (brand, columns in order, bottom bar, next/link vs anchor, fallback bar) (AC1, AC4, AC5)
- [ ] T7 Implement `components/site-footer/` (component, texts, styles, barrel)
- [ ] T8 Render `<SiteFooter />` in the locale layout; remove My Account's footer bar and its strings/styles; update affected tests; Changelog rows on account-signup-login (layout, index.ts) and account-profile (AC14 superseded) (AC1, AC7)

## Verification

- [ ] T9 Full verify (dev server stopped for the build): frontend vitest/tsc/lint/build, cms tsc/test
- [ ] T10 Live browser check: footer on Home, catalog, a PDP and My Account (one footer); links navigate; 1280/760/375; fallback bar with the footer document missing; an admin edit shows within 60s (AC1, AC5, AC6)
- [ ] T11 QA against AC1-AC7 (qa skill) → qa-report.md

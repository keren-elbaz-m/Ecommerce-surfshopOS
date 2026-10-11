# Tasks — Catalog Listing Page

> Epic: storefront-catalog
> Spec: 2026-09-29-catalog-listing-page
> Branch: feat/storefront-catalog/catalog-listing-page

## URLs + data

- [x] T1 Failing tests in `__tests__/lib/routes.test.ts` for `PRODUCTS_HREF` + `catalogHref` (query order, page 1 omitted, gender dropped off non-gendered categories, parity with existing builders)
- [x] T2 Implement `PRODUCTS_HREF` + `catalogHref` in `frontend/src/lib/routes.ts`
- [x] T3 Failing tests in `__tests__/lib/strapi/products.test.ts`: query (fields/populate, each filter only when set, Gender `$in`, `sort=id:asc`, page + pageSize 16), revalidate 60, maps via `toProductCard`, returns total/pageCount, `null` on network error / non-2xx / malformed body
- [x] T4 Implement `frontend/src/lib/strapi/products.ts` (`PAGE_SIZE`, `getProductList`)

## Page model

- [x] T5 Failing tests in `__tests__/components/catalog/catalog.test.ts`: `resolveCatalogParams` (AC3/AC4 incl. sub from another category, invalid gender ignored, page parsing, array params), `buildBreadcrumb` (AC6), `buildSidebar` (AC8: active/open, Clothing Shop All + Men/Women with All Men/Women, Wetsuits flat, `/products` nothing open), `buildPagination` (AC7: 17 → 2 pages, mocked 7 pages, Prev/Next presence, sub/gender kept)
- [x] T6 Implement `frontend/src/components/catalog/catalog.ts` + `catalog-texts.ts`

## UI

- [x] T7 Failing tests in `__tests__/components/catalog/CatalogPage.test.tsx` + `CatalogSidebar.test.tsx`: heading, breadcrumb, "N results", 16 cards with the first 4 priority, no heart / sort / filter / Finder, empty state + All <Category> link, unavailable state, `notFound` for unknown category/sub and out-of-range page, pagination links, sidebar toggles `aria-expanded` and active `aria-current`
- [x] T8 Add `chevron-down` to `shared/components/icons/Icon.tsx`; implement `CatalogSidebar.tsx` and `CatalogPagination.tsx`
- [x] T9 Implement `CatalogPage.tsx` + `index.ts`; add `app/[locale]/products/page.tsx` and `app/[locale]/products/category/[categorySlug]/page.tsx`

## Verification

- [x] T10 Full frontend suite (testing skill), `npm run lint`, `tsc --noEmit`; live curl checks against local Strapi. **2026-09-29:** 179/180 (only the pre-existing hero AC13 failure); `tsc` clean; lint clean (pre-existing layout font warning only). Live: `/products` 17 results, 16 cards, Page 1–2 + Next; `?page=2` 1 card + Prev; `?page=3|0|abc` 404; surfboards 8 (and `?gender=men` ignored, 8); clothing men 3 / women 2, All Men / All Women active; `?sub=shorts&gender=men` 1; accessories `?sub=traction` empty state; unknown category and `surfboards?sub=fins` 404
- [ ] T11 Run QA against acceptance criteria (qa skill), incl. a browser check at desktop, 1080, 900 and mobile widths against catalog.html

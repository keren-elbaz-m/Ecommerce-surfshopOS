# Tasks — Surfboard Detail Page

> Epic: storefront-catalog
> Spec: 2026-10-06-surfboard-detail-page
> Branch: feat/storefront-catalog/surfboard-detail-page

## Data

- [x] T1 Failing tests in `__tests__/lib/strapi/product.test.ts`: query (Slug filter, fields, Images/Category/BoardSizes/SurfboardSpecs.Video populate, pageSize 1), revalidate 60, `ok` / `not-found` (empty data) / `unavailable` (network error, non-2xx, malformed body)
- [x] T2 Implement `frontend/src/lib/strapi/product.ts` (`getProductBySlug`, `StrapiProductDetail`)

## View model

- [x] T3 Failing tests in `__tests__/components/product-detail/surfboard-detail.test.ts`: AC3 null cases; sizes (sort, sold out, default index, all sold out, none); 12 attribute items in section order with labels + scales; exact values 34 → 34, 1 → 1, 100 → 100, non-integer unrounded; guard clamp 0 → 1, 150 → 100; Skill Beginner/Intermediate/Advanced → 1/50/100; no SurfboardSpecs → no attributes; video on/off; breadcrumb
- [x] T4 Implement `components/product-detail/surfboard-detail.ts` + `product-detail-texts.ts`

## UI

- [x] T5 Add `react-markdown` to `frontend/package.json`; add Icon shapes `close`, `return`, `clock` (shipping and read-more reuse the existing `cart` and `arrow-right`) to `shared/components/icons/Icon.tsx`
- [x] T6 Failing tests for `SurfboardGallery`, `SurfboardBuyPanel`, `SurfboardDescription` (AC7–AC9, AC11)
- [x] T7 Implement `SurfboardGallery.tsx` (gallery + lightbox), `SurfboardBuyPanel.tsx`, `SurfboardDescription.tsx`
- [x] T8 Failing tests in `__tests__/components/product-detail/ProductDetailPage.test.tsx`: AC3–AC6; AC10 exact rendering — value 34 → fill `width: 34%`, marker `left: 34%`, `aria-valuenow="34"`, `aria-valuemin="1"`, `aria-valuemax="100"`, plus the same for 1 and 100, and scale labels at start/middle/end; AC12 incl. no fin setup renders
- [x] T9 Implement `ProductDetailPage.tsx` + `index.ts`; add `app/[locale]/products/[productSlug]/page.tsx`

## Verification

- [x] T10 Full frontend suite (testing skill), `npm run lint`, `tsc --noEmit`; live checks against local Strapi: `/products/tideline-6-0-performance-shortboard` 200 with 4 sizes, `/products/ci-pro` shows 6'6" sold out, a clothing slug 404, `/products/nope` 404, `/products/category` 404, `/products` + `/products/category/surfboards` still the catalog. **2026-10-06:** 270/270 tests; `tsc` clean; lint clean (pre-existing layout font warning only). Live: tideline 200 with 4 sizes; ci-pro 200 with 6'6" · 40.6L disabled "— Sold out"; `/products/nope`, `/products/category` and `/products/samurai-pro-22-boardshort` 404; `/products` and `/products/category/surfboards` 200. All 12 meters on tideline and ci-pro match Strapi exactly (aria-valuenow, fill width, marker left); no fin text
- [x] T11 Run QA against acceptance criteria (qa skill), incl. a browser check at desktop, 1100, 900 and mobile widths against `product-surfboard-tideline.html`, and a visual check on a seeded board that each of the 12 markers sits exactly at its value (compare marker centre / track width to the Strapi value) **2026-10-06:** PASS WITH NOTES — see [qa-report.md](qa-report.md). Browser at 1440/1100/900/390: grid matches design at every width; 96 meters (2 boards × 4 widths) max marker/fill error 0.004 %, labels 0.00 px off their 0/50/100 % points; lightbox focus/scroll-lock/Esc OK

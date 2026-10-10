# Tasks — Standard Product Detail Page

> Epic: storefront-catalog
> Spec: 2026-10-07-standard-product-detail-page
> Branch: feat/storefront-catalog/standard-product-detail-page

## Data

- [x] T1 Failing tests in `__tests__/lib/strapi/product.test.ts`: query also has `fields[5]=Gender` and `populate[StandardSizes]=true`; ok result passes StandardSizes + Gender through (AC2)
- [x] T2 Implement in `lib/strapi/product.ts`: query additions, `Gender` and `StandardSizes` on `StrapiProductDetail`

## Shared extraction (no surfboard behaviour change)

- [x] T3 Run the surfboard suite as a baseline; write failing tests `__tests__/components/product-detail/shared/breadcrumb.test.ts` (incl. gender level: Men/Women on clothing; none for Unisex, no Gender, or non-gendered category) and `PhotoLightbox.test.tsx` (dialog, Esc/backdrop/close → onClose, arrows wrap via onIndexChange, scroll lock, focus to close)
- [x] T4 Implement `shared/breadcrumb.ts`, `ProductBreadcrumb.tsx`, `PhotoLightbox.tsx`, `TrustList.tsx`, `ProductUnavailable.tsx`; switch `SurfboardGallery` to `PhotoLightbox` and `SurfboardBuyPanel` to `TrustList`
- [x] T5 Move the surfboard files into `surfboard/` and cut the grid out of `ProductDetailPage` into `surfboard/SurfboardDetail.tsx`; update surfboard test imports only; the surfboard suite must be green and unchanged otherwise

## View model

- [x] T6 Failing tests `standard/standard-detail.test.ts`: null cases (Surfboard, no images, no category, NaN price); size order S/M/L/XL/OneSize; One Size label; soldOut / lowStock (0 → sold out, 1 and 3 → low, 4 → normal); default index; all sold out / no sizes → -1; gender only on clothing Men/Women
- [x] T7 Implement `standard/standard-detail.ts` + `standard` block in `product-detail-texts.ts`; add the `danger` colour token (`#C0392B`) to `frontend/tailwind.config.ts`

## UI

- [x] T8 Failing tests: `StandardGallery.test.tsx` (AC7–AC9: one button per photo with labels, priority first, single photo spans, click opens lightbox at that index, focus returns to that photo); `StandardBuyPanel.test.tsx` (AC11–AC13: order, aria-pressed, Size — label updates, sold-out disabled + name, low-stock flag + note on select, all sold out → disabled "Sold out"); `StandardAccordions.test.tsx` (AC15: Description open by default, one open at a time, aria-expanded/controls, markdown without raw HTML)
- [x] T9 Implement `StandardGallery.tsx`, `StandardBuyPanel.tsx`, `StandardAccordions.tsx`
- [x] T10 Failing tests in `ProductDetailPage.test.tsx`: replace "404s for a non-surfboard product" with "renders a Standard product" (H1, eyebrow, price, size grid, trust list, accordions, gender breadcrumb); 404 for a Standard product missing a required field and for an unknown SizeType; AC16 hidden items (no Size Chart, Color, wishlist, "You might also like", Subtitle pill); a single `getProductBySlug` call
- [x] T11 Implement `standard/StandardProductDetail.tsx` and the SizeType switch in `ProductDetailPage.tsx`

## Docs

- [x] T12 surfboard-detail-page `spec.md`: Changelog row for the SurfboardDetail extraction / shared folder, plus the Overview and Out of Scope 404 lines updated to point at this spec

## Verification

- [x] T13 Full frontend suite (testing skill), `npm run lint`, `npx tsc --noEmit`; live against local Strapi: `/products/samurai-pro-22-boardshort` 200 with 6 photos, S–XL and breadcrumb Home / Clothing / Men; `/products/coldwater-4-3-chest-zip` S low-stock flag; `/products/horizon-snapback-cap` One Size; surfboards (ci-pro, mikey-february-s-shorty) unchanged; `/products/nope` and `/products/category` 404. **2026-10-07:** 352/352 tests (40 files; 287 before, all surfboard tests unchanged except imports and the one replaced 404 test); `tsc` clean; lint clean (pre-existing layout font warning only). Live: samurai 200, 6 photos, S–XL with S selected, breadcrumb Home / Clothing / Men / name; coldwater S flagged "S — Low stock" and the note shown (S is the default); cap "One Size"; archive-team-tee (Unisex) has no gender level and XL is flagged; ci-pro and mikey-february-s-shorty 200 with 12 meters and the Dimensions select (1 sold-out option each); `/products/nope` and `/products/category` 404; `/products` and `/products/category/surfboards` 200. A one-off Tailwind build confirms `bg-danger`, `text-danger`, the scroll-snap and the 420px grid classes compile. The running `next dev` needs a restart to pick up the new `danger` token
- [x] T14 Run QA against acceptance criteria (qa skill), incl. a browser check at 1440, 900, 420 and 390 against `product-apparel-samurai-boardshort.html`: sticky buy column, 2-up grid, mobile scroll-snap swipe, no layout shift as photos load, lightbox; plus a surfboard regression pass on ci-pro and mikey-february-s-shorty at 1440 and 390 **2026-10-07:** PASS WITH NOTES — see [qa-report.md](qa-report.md). Grid, 2-up frames, H1 sizes and size-grid columns match the design at all 4 widths; mobile scroll-snap swipe settles on whole photos (touch swipe moves exactly one); frames identical with images blocked vs loaded; lightbox opens at the clicked photo and returns focus to it; coldwater S low-stock flag and note `#C0392B` after the dev-server restart; ci-pro and mikey unchanged (columns, 12 meters ≤0.0042 % error, sold-out options, lightbox)

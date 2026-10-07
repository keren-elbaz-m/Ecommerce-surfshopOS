# References for Standard Product Detail Page

## Related Specs

<!-- Populated by /shape-spec Step 4 (Pre-Flight Integrity Check). -->

- **[Related]** [2026-10-06-surfboard-detail-page](../2026-10-06-surfboard-detail-page/spec.md). It owns `/products/*`, `frontend/src/components/product-detail/**` and `frontend/src/lib/strapi/product.ts`. This spec:
  - moves its 3-column markup into `product-detail/surfboard/SurfboardDetail.tsx`, with no behaviour change
  - extracts the breadcrumb, lightbox, trust list and unavailable state into `product-detail/shared/`
  - adds `Gender` and `StandardSizes` to the detail query

  It is kept separate because the layouts, view models, galleries and buy panels differ, and only the page chrome is shared. One surfboard test is replaced on purpose: "404s for a non-surfboard product".
- **[Related]** [2026-09-28-product-content-model](../2026-09-28-product-content-model/spec.md). The `StandardSizes` component (`Size`: S / M / L / XL / OneSize, `Stock`) and `Gender` (Men / Women / Unisex). Strapi is unchanged.
- **[Related]** [2026-09-28-product-card](../2026-09-28-product-card/spec.md). `formatPrice` and the `{ src, width, height }` image shape. Every card links to `/products/<slug>`, which now resolves for standard products too.
- **[Related]** [2026-09-29-button-cta](../2026-09-29-button-cta/spec.md). Add to Cart uses its button mode. `disabled`, used for "Sold out", is already supported (`disabled:bg-muted`, button-cta AC9), so no change is needed.
- **[Related]** add-to-cart (not yet shaped). It will wire the inert Add to Cart button in `StandardBuyPanel`.

## Shared config

- `frontend/tailwind.config.ts` gains a `danger` colour token (`#C0392B`, the design's low-stock red). It is used as `bg-danger` / `text-danger` for the low-stock flag and note. No spec owns this file; the change is recorded here and in this spec's Changelog.

## Similar Implementations

### Surfboard detail page

- **Location:** `frontend/src/components/product-detail/**`, `frontend/src/__tests__/components/product-detail/**`
- **Relevance:** the sibling page, and the source of the shared chrome.
- **Key patterns:**
  - an async server page that maps `getProductBySlug` results to `notFound()` or the unavailable state
  - a pure view model next to the component
  - gallery and lightbox state in one client component
  - the test mocks for `@/lib/strapi/product` and `next/navigation`

### Sizes

- **Location:** `frontend/src/lib/strapi/sizes.ts`
- **Key patterns:** the `StrapiStandardSize` type and `formatStandardSize` (`OneSize` → "One Size").

### Routes

- **Location:** `frontend/src/lib/routes.ts`
- **Key patterns:** `genderHref(categorySlug, 'men' | 'women')`, `GENDERED_CATEGORY_SLUGS = ['clothing']` and the `StrapiGender` type.

## Seed data

- `samurai-pro-22-boardshort`: 6 photos (700×874), S:16 M:24 L:24 XL:12, Men, clothing / boardshorts.
- `coldwater-4-3-chest-zip`: S:1, which exercises low stock.
- `horizon-snapback-cap`: OneSize:15, accessories, no gender.
- `archive-team-tee`: Unisex, XL:2 (low).
- Standard products other than samurai use generated placeholder images.
- Surfboard regression boards: `ci-pro` (a sold-out size) and `mikey-february-s-shorty`. Tideline was deleted.

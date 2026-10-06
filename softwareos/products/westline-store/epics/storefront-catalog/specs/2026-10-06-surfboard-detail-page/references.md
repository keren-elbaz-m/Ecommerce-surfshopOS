# References for Surfboard Detail Page

## Related Specs

<!-- Populated by /shape-spec Step 4 (Pre-Flight Integrity Check). -->

- **[Considered]** [2026-09-29-catalog-listing-page](../2026-09-29-catalog-listing-page/spec.md). It matched on `/products` ⊃ `/products/*` (strong, route prefix) and was dismissed in Step 4d as a different concern. It owns the listing at `/products` and `/products/category/*`, and its Out of Scope explicitly defers "the PDP at `/products/[slug]`" to this spec. In Next 14.2 the static `category` segment wins over `[productSlug]`, so the routes don't collide.
- **[Related]** [2026-09-28-product-card](../2026-09-28-product-card/spec.md). Its AC7 links every card to `/products/<slug>`, which this spec serves. This spec reuses `formatPrice` from `product-card.ts` and the `{ src, width, height }` image shape.
- **[Related]** [2026-09-28-product-content-model](../2026-09-28-product-content-model/spec.md). This spec reads its Product schema: `SizeType`, `BoardSizes` (`product.board-size`) and `SurfboardSpecs` (`product.surfboard-specs`). `SkillLevel` is an enum (Beginner / Intermediate / Advanced). The other 11 attributes are integers with schema `min: 0`, `max: 100`, and this spec clamps them to 1–100 for display.
- **[Related]** add-to-cart (not yet shaped). It will wire the inert Add to Cart button in `SurfboardBuyPanel`.

## Similar Implementations

### Catalog listing page

- **Location:** `frontend/src/components/catalog/**`, `frontend/src/app/[locale]/products/**`
- **Relevance:** the closest sibling. It is an async server page under the same `/products` tree.
- **Key patterns:**
  - `CatalogPage.tsx`: `notFound()` for invalid input and an unavailable state (200) when Strapi fails.
  - The `CONTAINER` and breadcrumb Tailwind classes.
  - `catalog.ts`: a pure page model.
  - `catalog-texts.ts`.
  - `CatalogSidebar.tsx`: client state in one component.
  - One-line route wrappers.

### Strapi data access

- **Location:** `frontend/src/lib/strapi/products.ts`, `frontend/src/lib/strapi/client.ts`
- **Relevance:** the model for `lib/strapi/product.ts`.
- **Key patterns:**
  - Queries are built with `URLSearchParams`, using explicit `fields[i]` and `populate[...]`.
  - `strapiFetch(path, { label, fallback: null })` never throws.
  - Responses are cast to `Raw…` types and shape-checked, with `null` on a malformed body.
  - Tests: `__tests__/lib/strapi/products.test.ts` (`// @vitest-environment node`, `vi.stubGlobal('fetch')`, a `strapiResponse` helper).

### Product card

- **Location:** `frontend/src/shared/components/product-card/**`, `frontend/src/lib/strapi/media.ts`
- **Relevance:** the image mapping and the client-side gallery state.
- **Key patterns:**
  - `strapiMediaUrl` for `/uploads/…` URLs.
  - `formatPrice` (USD, no decimals for whole numbers).
  - `ProductCardMedia.tsx` (`'use client'`, index state, `next/image` with `priority` on the first image).

### Sizes

- **Location:** `frontend/src/lib/strapi/sizes.ts`
- **Relevance:** the Dimensions select.
- **Key patterns:** the `StrapiBoardSize` type, `formatBoardSize` (`{ LengthFt: 6, LengthInches: 0, VolumeL: 29.4 }` → `6'0" · 29.4L`) and `boardLengthInches` for ordering.

### Icons

- **Location:** `frontend/src/shared/components/icons/Icon.tsx` (owned by site-nav)
- **Relevance:** all icons go through `<Icon name>`. This spec adds `close`, `return` and `clock`, and reuses `cart` (shipping), `arrow-right` (read-more chevron), `arrow-left` and `chevron-down`.

## Seed data

- `cms/seed/catalog/catalog.json` has 13 surfboards, all with BoardSizes and SurfboardSpecs. None has a Video.
- `tideline-6-0-performance-shortboard` has 4 sizes and a markdown blockquote Description.
- `ci-pro` has its 78" (6'6") size at Stock 0, which exercises the sold-out option.

### Button CTA

- **Location:** `frontend/src/shared/components/button-cta/` (owned by button-cta)
- **Relevance:** Add to Cart / Sold out use its button mode (no `href`, `disabled`).

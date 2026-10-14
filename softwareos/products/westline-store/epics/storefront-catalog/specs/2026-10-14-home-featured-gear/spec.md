# Home Featured Gear

> Epic: ../../epic.md
> Status: in-progress
> Branch: feat/storefront-catalog/home-featured-gear
> Covers: frontend/src/components/home/featured-gear/**, frontend/src/__tests__/components/home/featured-gear/**, frontend/src/shared/components/product-slider/**, frontend/src/__tests__/shared/components/product-slider/**, frontend/src/components/product-detail/shared/RelatedProducts.tsx

## Overview

The Home page's "Featured gear" section, below the hero. It shows the products marked `Featured` in Strapi as the shared product card (with the wishlist heart), in a snap slider. Its eyebrow and title are editable on the Homepage single type, like the hero. Design: `westline-site-desing/index.html` `section.featured` (see visuals/featured-gear-notes.md). This is the Home section that home-hero and product-card deferred ("Featured Gear… separate specs").

Decisions (2026-10-14):
- **Which products:** all `Featured` products, newest first, up to 8.
- **Slider at every width:** 4 cards per view on desktop, with arrows when there are more than 4. The design's desktop grid has no arrows; this is a small, recorded extension.
- **Section copy:** editable on the Homepage (`FeaturedEyebrow`, `FeaturedTitle`), with "New in" / "Featured gear" as the seed values and the frontend fallback.

## Goals & User Stories

- As a shopper on Home, I see a few hand-picked products and can open or save them.
- As a content editor, I choose what's featured by toggling a product's `Featured` flag, and I can change the section's eyebrow and title.

## Acceptance Criteria

1. AC1: Below the hero, Home shows a "Featured gear" section.
   - The eyebrow and title come from the Homepage (`FeaturedEyebrow` / `FeaturedTitle`), falling back to "New in" / "Featured gear" when empty.
   - Its cards are the products with `Featured` = true, newest first, up to 8, rendered on the server.
2. AC2: The cards are the shared product card with the wishlist heart (`WishlistProductCard`), so saving works as in the catalog.
3. AC3: The section is a slider:
   - 4 cards per view above 980px, 2 at ≤980px, 1 at ≤520px, with snap scrolling.
   - Prev/next arrows ("Previous products" / "Next products") are hidden at either end and when nothing overflows.
   - There's one dot per snap position ("Go to slide N", with `aria-current` on the current one). The dots are hidden when there are fewer than 2 positions.
4. AC4: With no featured products (or Strapi down), the section isn't rendered and the page still loads.
5. AC5: Layout matches the design at 1280, 900 and 375 with no page-level horizontal scroll.
6. AC6: Both product pages end with the design's "You might also like" rail (`.related`: 64px above, a `clamp(22px,3vw,30px)` display heading), holding the Featured products minus the product being viewed, in the same slider and cards as Home. Hidden when nothing else is featured, when Strapi fails, and on the unavailable state. *(Added 2026-10-14, Ori's request.)*

## Technical Approach

**CMS** (home-hero's `homepage`):
- Add optional `FeaturedEyebrow` and `FeaturedTitle` (string) to `homepage/schema.json`.
- `seed/homepage/homepage.json` and `seed-homepage.js` set them for new databases. Existing databases keep the create-only rule, and the frontend fallback covers them.

**Frontend:**
- `lib/strapi/products.ts` (catalog-listing-page's file): add `getFeaturedProducts(limit = 8)`, querying `filters[Featured][$eq]=true` sorted `updatedAt:desc`. It uses the same fields as the product list and maps with `toProductCard`.
- `lib/strapi/homepage.ts` (home-hero's file): add `getHomepageFeaturedText()`, returning `{ eyebrow, title }` or `null`.
- `components/home/featured-gear/`:
  - `FeaturedGear.tsx`: async server component. It fetches both and returns `null` when there are no products.
  - The track is the shared `shared/components/product-slider` (`ProductSlider` + `useProductSlider`, cards passed as children; `stepCount()` tested), also used by the product pages.
  - `featured-gear-texts.ts`, `featured-gear.styles.ts`, `index.ts`.
- `app/[locale]/[[...segments]]/page.tsx` (home-hero's file): renders `<Hero />` then `<FeaturedGear />`.

## Out of Scope

- The other Home sections: the Surfboard Finder teaser, trust marquee and newsletter.
- An add-to-cart button on the cards (cards have none, per product-card).
- Manual ordering of featured products.

## Standards Applied

- [frontend/components](../../../../../../standards/frontend/components.md): one component and one return per file; texts file; `ProductCard` reused through `WishlistProductCard`.
- [global/git-workflow](../../../../../../standards/global/git-workflow.md): branch `feat/storefront-catalog/home-featured-gear`, stacked on site-footer.
- [global/tech-stack](../../../../../../standards/global/tech-stack.md): no new dependencies.

## Changelog

| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-10-14 | Ori Chai Matan | created | Initial shaping | — |
| 2026-10-14 | Ori Chai Matan | change | Implemented on its branch (commit `9332825`). The slider is a client component with `useFeaturedSlider` (`stepCount` = the design's `positions()`); on desktop the arrow padding applies only when more than 4 products overflow, so ≤4 keeps the design's full-width row. QA: PASS WITH NOTES | qa-report.md |
| 2026-10-14 | Ori Chai Matan | change | Ori's request: Featured products also show on the product pages as the design's "You might also like" rail (new AC6; `components/product-detail/shared/RelatedProducts.tsx`, fetched in `ProductDetailPage` with the product). To share it, the slider moved to `shared/components/product-slider` (`ProductSlider` takes the cards as children so it doesn't depend on the wishlist). Covers extended. Verified live on both product types (self excluded) and Home; 893 frontend tests | commit `5ec5af2` |

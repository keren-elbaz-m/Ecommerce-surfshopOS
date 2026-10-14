# References for Product Card

## Related Specs

- **[Related]** [2026-09-28-product-content-model](../2026-09-28-product-content-model/spec.md) is the data source. `toProductCard` maps its REST Product (`Name`, `Slug`, `Price`, `Images`, `Category.Slug`). The two specs stay separate because the model is backend-only and the card is a frontend unit.
- **[Related]** customer-account-wishlist (epic) owns the favorites logic that feeds `isFavorite` and `onToggleFavorite`. The card only renders the heart when both are passed.

The cross-tree coverage search in Step 4 found no spec claiming these paths, and no spec body mentions a product card.

## Upstream Meeting

_None. No meeting write-ups exist._

## Similar Implementations

### Hero carousel (client island + dots)

- **Location:** `frontend/src/components/home/hero/hero-carousel.tsx`, `frontend/src/__tests__/components/home/hero/hero-carousel.test.tsx`
- **Relevance:** it's the only existing client island. It has `aria-current` dots and uses `next/image` with remote Strapi uploads.
- **Key patterns:** dots as `<button aria-label aria-current>`; `motion-reduce:` variants; RTL tests with real `next/image`.

### Strapi media URL

- **Location:** `frontend/src/lib/strapi/homepage.ts` (`absoluteUrl`)
- **Relevance:** it's the origin of the shared `strapiMediaUrl` in `frontend/src/lib/strapi/media.ts`. `homepage.ts` switches to importing it.

### Remote images

- **Location:** `frontend/next.config.mjs`
- **Relevance:** its `images.remotePatterns` already allows Strapi `/uploads/**`, so `next/image` works with no config change.

## Visuals

The design files are referenced by path and not copied:
- `westline-workplace/westline-site-desing/index.html`: the Featured gear `.prod-card`. This is the canonical card.
- `westline-workplace/westline-site-desing/catalog.html`: an older, simpler card, superseded.

`visuals/card-notes.md` has the extracted CSS, markup and behavior.

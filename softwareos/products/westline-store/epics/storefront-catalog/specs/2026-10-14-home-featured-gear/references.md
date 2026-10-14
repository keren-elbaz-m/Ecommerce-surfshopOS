# References for Home Featured Gear

## Related Specs

- **[Related]** [2026-09-27-home-hero](../2026-09-27-home-hero/spec.md): owns the Home page, `lib/strapi/homepage.ts` and the homepage single type; this spec adds two optional fields and the section after the hero.
- **[Related]** [2026-09-28-product-card](../2026-09-28-product-card/spec.md): the card ("the Home Featured gear section … renders"), unchanged.
- **[Related]** [2026-09-29-catalog-listing-page](../2026-09-29-catalog-listing-page/spec.md): owns `lib/strapi/products.ts`; this spec adds `getFeaturedProducts`.
- **[Related]** [customer-account-wishlist / 2026-10-14-wishlist](../../../customer-account-wishlist/specs/2026-10-14-wishlist/spec.md): `WishlistProductCard`, which its notes say Featured gear should use when built.

## Similar Implementations

- `frontend/src/components/wishlist/use-wishlist-slider.ts` + `WishlistSection.tsx`: a scroll-snap slider with edge-aware arrows.
- `frontend/src/lib/strapi/products.ts` `getProductList`: the field and populate query and `toProductCard` mapping.

## Visuals

- `visuals/featured-gear-notes.md`

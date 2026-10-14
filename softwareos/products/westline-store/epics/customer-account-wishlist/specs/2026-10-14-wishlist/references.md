# References for Wishlist

## Related Specs

<!-- Planned small edits in files other specs own. The Changelog row on each owner is written when the edit is made, not at shaping. -->

- **[Related]** [2026-09-22-account-signup-login](../2026-09-22-account-signup-login/spec.md): owns `frontend/src/app/[locale]/layout.tsx`. This spec adds `WishlistProvider signedIn={…}` beside `CartProvider`. The auth flow itself is reused unchanged: `loginHref(next)`, the login page's register link carrying `next`, and `lib/auth/next-path.ts`.
- **[Related]** [2026-10-13-account-profile](../2026-10-13-account-profile/spec.md): owns `components/account/AccountOverview.tsx` and `app/[locale]/account/page.tsx`. `AccountOverview` renders `WishlistSection` instead of its static wishlist half, and the page fetches `getWishlist(jwt)` server-side. Its empty-state copy, the `#wishlist` hash routing and `SECTION_IDS.wishlist` are reused. This branch is stacked on account-profile's.
- **[Related]** [storefront-catalog / 2026-09-29-catalog-listing-page](../../../storefront-catalog/specs/2026-09-29-catalog-listing-page/spec.md): owns `components/catalog/CatalogPage.tsx`. It renders `WishlistProductCard` instead of `ProductCard`, which un-hides the `.prod-wish` heart that spec deferred to this epic.
- **[Related]** [storefront-catalog / 2026-10-07-standard-product-detail-page](../../../storefront-catalog/specs/2026-10-07-standard-product-detail-page/spec.md): owns `components/product-detail/standard/StandardBuyPanel.tsx`. It gains `.pdp-actions` with `WishlistButton variant="icon"` (the wishlist control that spec hid).
- **[Related]** [storefront-catalog / 2026-10-06-surfboard-detail-page](../../../storefront-catalog/specs/2026-10-06-surfboard-detail-page/spec.md): owns `components/product-detail/surfboard/SurfboardBuyPanel.tsx`. It gains `.pdp-actions-row` with `WishlistButton variant="labelled"`. Share stays unbuilt.
- **[Related]** [storefront-catalog / 2026-09-29-site-nav](../../../storefront-catalog/specs/2026-09-29-site-nav/spec.md): owns `frontend/src/lib/routes.ts`. It gains `WISHLIST_API_HREF` and `wishlistItemApiHref(slug)`. The nav heart (`WISHLIST_HREF`) is unchanged, with no badge.
- **[Related]** [storefront-catalog / 2026-09-28-product-card](../../../storefront-catalog/specs/2026-09-28-product-card/spec.md): **not modified.** Its AC8 controlled heart (`isFavorite` / `onToggleFavorite(slug)`) is consumed as built. Its 36px bordered heart matches the canonical 28 Sep designs.

## Upstream Meeting

_None._

## Similar Implementations

### Custom, owner-only Strapi routes

- **Location:** `cms/src/api/account/` (controller `requireUserId`, `AccountError` → status map), `cms/src/api/cart/` (`identity.ts` `createIdentify`, `auth: false` routes, path params for DELETE).
- **Relevance:** the same "only the session's own user, no core CRUD" rule (AC11).

### Layout-level client provider

- **Location:** `frontend/src/components/cart/` (`CartProvider.tsx`, `cart-context.ts`, server-safe `index.ts` barrel), wired in `app/[locale]/layout.tsx` with `signedIn={getSession() !== null}`.
- **Relevance:** `WishlistProvider` follows the same shape and wiring.

### Cookie-gated route handler

- **Location:** `frontend/src/app/api/account/profile/route.ts`, `lib/strapi/account.ts` `updateMe` (typed result union).
- **Relevance:** the shape for `app/api/wishlist/**` and `lib/strapi/wishlist.ts`.

### Card mapper

- **Location:** `frontend/src/shared/components/product-card/product-card.ts` (`toProductCard`).
- **Relevance:** `getWishlist` maps Strapi products to cards with it, so the wishlist renders the same card as the catalog.

## Visuals

- `visuals/wishlist-notes.md`: design → spec mapping and the gap list, from `westline-site-desing/` (28 Sep). The HTML files aren't copied (0.6–8.8 MB).

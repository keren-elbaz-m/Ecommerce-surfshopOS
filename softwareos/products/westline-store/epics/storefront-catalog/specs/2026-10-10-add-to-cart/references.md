# References for Add to Cart

## Related Specs

- **[Related]** [2026-10-06-surfboard-detail-page](../2026-10-06-surfboard-detail-page/spec.md): its Add to Cart (`SurfboardBuyPanel`) is wired here. Kept separate because the cart is its own feature with its own API and UI; the PDP only calls `useCart().add`.
- **[Related]** [2026-10-07-standard-product-detail-page](../2026-10-07-standard-product-detail-page/spec.md): AC13 (inert Add to Cart) is superseded. `StandardBuyPanel` is wired here, and its shipping copy now reads `FREE_SHIPPING_THRESHOLD`.
- **[Related]** [2026-09-29-site-nav](../2026-09-29-site-nav/spec.md): owns `SiteNav.tsx`, `lib/strapi/client.ts`, `lib/routes.ts` and icons. This spec swaps the cart Link for `CartNavButton`, adds `strapiRequest`, and adds `plus` / `minus` icons. Kept separate because nav structure stays with site-nav.
- **[Related]** [customer-account-wishlist/2026-09-22-account-signup-login](../../../customer-account-wishlist/specs/2026-09-22-account-signup-login/spec.md): owns `app/[locale]/layout.tsx` and `app/api/auth/**`. This spec adds `CartProvider` to the layout and a one-line guest-cart merge after the session is set in four auth routes, with no restructuring (frontend/components auth exception).
- **[Related]** [2026-09-28-product-content-model](../2026-09-28-product-content-model/spec.md): owns `lib/strapi/sizes.ts` and the product schema. This spec adds the size-key helpers; the Product schema is untouched, and the Cart relates to Product.
- **[Related]** cart-checkout epic, candidate `persistent-cart`: delivered by this spec. The checkout specs will consume the cart API (`GET /api/carts/current`).

## Design

- `westline-site-desing/cart.html` (notes in [visuals/cart-notes.md](visuals/cart-notes.md)): page layout, summary and empty state.
- `westline-site-desing/product-surfboard-tideline.html` around lines 939–960 and `product-apparel-samurai-boardshort.html` around lines 875–896: the cart drawer markup and JS.
- `westline-site-desing/promo-video/public/textures/live/pdp-cart-drawer-full.png` (not copied): the drawer screenshot.
- Design inconsistencies resolved in shaping:
  - The threshold differs across sources (texts $75, drawer JS 100, cart JS 900); resolved to $75.
  - The design shows line totals only, so this spec does the same.
  - The design keeps the cart in memory only; this spec persists it.

## Similar Implementations

### Product detail buy panels
- **Location:** `frontend/src/components/product-detail/{surfboard/SurfboardBuyPanel.tsx, standard/StandardBuyPanel.tsx}`
- **Relevance:** the Add to Cart call sites; they hold the selected-size state.
- **Key patterns:** client panel with lifted state, `ButtonCTA` with `disabled`, texts functions.

### PhotoLightbox
- **Location:** `frontend/src/components/product-detail/shared/PhotoLightbox.tsx`
- **Relevance:** the dialog pattern for the drawer.
- **Key patterns:** Esc / backdrop close, scroll lock, focus to close, parent restores focus.

### Auth route handlers
- **Location:** `frontend/src/app/api/auth/*/route.ts`, `frontend/src/lib/auth/session.ts`
- **Relevance:** the cookie-owning proxy pattern for `app/api/cart/**`; also the merge call sites.
- **Key patterns:** httpOnly cookie options helper, `NextResponse.cookies.set`.

### Strapi client and resources
- **Location:** `frontend/src/lib/strapi/{client,product,sizes}.ts`
- **Relevance:** where `strapiRequest`, `cart.ts` and the size keys go.
- **Key patterns:** never-throw fetch, shape checks, `formatBoardSize` / `formatStandardSize`.

### Strapi product API + document middleware
- **Location:** `cms/src/api/product/**`, `cms/src/api/product/documents/clear-stale-sizes.ts`
- **Relevance:** explains why size component ids aren't stable, so the cart uses derived SizeKeys.

### Formatters
- `formatPrice` (`frontend/src/shared/components/product-card/product-card.ts`), `ButtonCTA`, `Icon`, `CONTAINER` (`product-detail/shared/layout.ts`), the `bg-background` / `danger` tokens.

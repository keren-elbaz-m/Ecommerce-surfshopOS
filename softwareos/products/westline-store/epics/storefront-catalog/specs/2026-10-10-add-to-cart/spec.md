# Add to Cart

> Epic: ../../epic.md
> Status: done
> Branch: feat/storefront-catalog/add-to-cart
> Covers: /cart, frontend/src/app/[locale]/cart/**, frontend/src/app/api/cart/**, frontend/src/components/cart/**, frontend/src/lib/strapi/cart.ts, frontend/src/lib/cart/**, frontend/src/lib/shipping.ts, cms/src/api/cart/**, cms/src/components/cart/**, cms/tests/cart/**, frontend/src/__tests__/components/cart/**, frontend/src/__tests__/lib/strapi/cart.test.ts, frontend/src/__tests__/lib/cart/**, frontend/src/__tests__/app/api/cart/**

## Overview

Wires the inert Add to Cart on both detail pages and builds the cart, closing the brief's Product → Cart leg (Product → Cart → Checkout).

**How it works**
- A shopper picks a size and clicks Add to Cart.
- The line is saved to a Strapi cart, and a right-side cart drawer opens.
- The header bag shows the item count and also opens the drawer.
- `/cart` is the full cart page, where shoppers can review lines, change qty, remove lines and see the total.

**Storage and identity**
- The cart lives in Strapi in a new `Cart` collection, so it survives reloads and devices and attaches to an account.
- Guests are identified by a random token in an httpOnly cookie.
- Signed-in shoppers are identified by their user.
- On login, the guest cart merges into the user's cart.

**Freshness**
- Price, stock, name and image are always read live from the product. A line stores only product + size + qty.
- A line can never show a stale price or exceed stock.

**Relation to the epics**
- This is storefront-catalog's `add-to-cart` candidate.
- It also delivers cart-checkout's `persistent-cart` candidate.
- Checkout stays disabled ("Checkout is coming soon.") until cart-checkout's checkout specs.

**Design:** `westline-site-desing/cart.html` (page) and the drawer on both product pages (see [visuals/cart-notes.md](visuals/cart-notes.md)).

## Goals & User Stories

- As a shopper, I add the size I picked to my cart and immediately see it in a cart drawer, so I know it worked.
- As a shopper, I see how many items are in my cart from any page, and I can open the cart from the header.
- As a shopper, I review my cart, change quantities or remove items, and see my subtotal and how close I am to free shipping.
- As a shopper, my cart is still there when I reload or come back later. When I sign in, the items I added as a guest are kept.
- As a shopper, I'm never shown a price or quantity the store can't honour. Sold-out items are clearly marked.

## Acceptance Criteria

1. **AC1: Strapi model.**
   - New collection `api::cart.cart` (draftAndPublish off) with:
     - `Token`: string, unique, private.
     - `User`: oneToOne → `plugin::users-permissions.user`, optional.
     - `Lines`: repeatable component `cart.line` with `Product` (manyToOne → product, required), `SizeKey` (string, required), `Quantity` (integer, min 1, required).
   - No core CRUD routes are exposed and no public permissions are granted.
   - Carts are visible in the admin Content Manager.
   - The Product schema is unchanged.
2. **AC2: Strapi cart API.** Custom routes (`auth: false`, identity resolved by the cart service):
   - `GET /api/carts/current`
   - `POST /api/carts/current/lines` `{ productDocumentId, sizeKey }`
   - `PATCH /api/carts/current/lines` `{ productDocumentId, sizeKey, quantity }`
   - `DELETE /api/carts/current/lines/:productDocumentId/:sizeKey`
   - `POST /api/carts/merge`

   Identity:
   - A valid `Authorization: Bearer <jwt>` → the user's cart.
   - Otherwise `x-cart-token` → the guest cart.
   - Otherwise no cart.
   - An invalid JWT → 401.
   - A token that belongs to a user's cart is ignored for guests.

   Request rules:
   - A GET with no cart returns an empty cart and creates nothing.
   - The first POST creates the cart (guest: a new `crypto.randomUUID()` token, returned in the response).
   - Bad bodies (missing ids, non-integer qty, qty < 1 or > 99) → 400.
   - An unknown product or sizeKey → 404.
3. **AC3: Line rules (server).**
   - Same product + same sizeKey = one line.
   - POST adds 1, clamped to that size's Stock.
   - PATCH sets qty, clamped to Stock.
   - Adding a sold-out size → 409 and the cart is unchanged.
   - DELETE removes the line. Removing the last line leaves an empty cart.
   - SizeKey format:
     - Board: `${LengthFt}-${LengthInches}-${Number(VolumeL)}`, e.g. `6-0-29.4`.
     - Standard: the `Size` enum (`S`…`OneSize`).
4. **AC4: Re-validation on every read.** Every cart response joins live, **published** product data (Name, Slug, Price, first Image, the size, Stock); `Lines.Product` is populated with `status: 'published'`. Per line:
   - **qty > Stock > 0:** qty is clamped and persisted, and the line is flagged `adjusted` in that response.
   - **Stock 0:** the line is kept and flagged `soldOut`.
   - **The SizeKey no longer exists on the product** (e.g. an admin edited LengthFt / LengthInches / VolumeL): the line is kept and flagged `unavailable`, a separate flag from `soldOut`.
   - **Product deleted or unpublished:** the line is pruned and persisted, and the response carries `removedCount > 0`. Today Product has `draftAndPublish: false`, so this only matters if drafts are turned on later.
   - The response never contains a stored price.
5. **AC5: Account merge.** When login, register, reset-password or Google OAuth completion sets the session, and a `westline_cart` cookie exists:
   - The Next route calls `POST /api/carts/merge` with the new JWT and the guest token.
   - Lines are summed per product + sizeKey and clamped to Stock.
   - The guest cart is deleted, and the response clears the `westline_cart` cookie.
   - A failed merge never fails the auth response; it is logged.
   - After logout the shopper gets a fresh guest cart on their next add. The user's cart stays with the account.
6. **AC6: Next proxy.**
   - Route handlers:
     - `GET /api/cart`
     - `POST|PATCH|DELETE /api/cart/lines`
   - Each reads `westline_session` (→ Bearer) and `westline_cart` (→ `x-cart-token`).
   - Calls go through `lib/strapi/cart.ts` on a new non-cached `strapiRequest` in `client.ts`.
   - A newly issued guest token is set as `westline_cart`: httpOnly, sameSite lax, secure in production, path `/`, maxAge 30 days.
   - Bodies are validated before forwarding.
   - Strapi down → 503 `{ error }`.
   - When Strapi rejects the session JWT (401 with a `westline_session` cookie), the call is repeated as a guest, `westline_session` is cleared and the response carries `x-westline-session: expired`; the provider then calls `router.refresh()` so the header shows Sign In.
   - Responses are the frontend `CartView` (see AC7) and `no-store`.
7. **AC7: Cart view.** `toCartView` maps the Strapi response to:
   - **Lines:** key, name, href (`productHref`), size label (`formatBoardSize` / `formatStandardSize`; for an `unavailable` line, the label stored from the key), unit price, line total (`formatPrice`), qty, maxQty (Stock), image, `soldOut`, `unavailable`, `adjusted`.
   - **Totals:** `count` (sum of qty over all lines), `subtotal` (sum over lines that are neither `soldOut` nor `unavailable`), `remainingForFreeShipping`, `freeShippingProgress` (0–1).
   - **Flags:** `hasBlockingLines` (any `soldOut` or `unavailable` line), `removedCount`.
8. **AC8: CartProvider.**
   - A `'use client'` provider in `app/[locale]/layout.tsx` wraps `SiteNav` and the page.
   - It loads `GET /api/cart` once on mount, and again whenever the session changes (the layout passes `signedIn`), so login merges and logout show without a full reload.
   - It exposes the cart, status (`loading | ready | error`), `drawerOpen`, `add`, `setQty`, `remove`, `openDrawer` and `closeDrawer` through `useCart()`.
   - Every mutation replaces the cart with the server response; there are no optimistic totals.
   - Only one mutation per line runs at a time.
9. **AC9: Add to Cart on the detail pages.**
   - `SurfboardBuyPanel` and `StandardBuyPanel` call `add(productDocumentId, sizeKey)` for the selected size.
   - While pending, the CTA is disabled and reads "Adding…".
   - On success the drawer opens.
   - On failure a `text-danger` note under the CTA reads "Couldn't add to cart. Try again." and the drawer stays closed.
   - A 409 (sold out meanwhile) shows "This size just sold out."
   - The sold-out / disabled behaviour of both panels is unchanged.
   - Mappers add `documentId` to both detail view models and `key` to each size.
   - `getProductBySlug` relies on the `documentId` that Strapi v5 always returns, so the query is unchanged apart from the type.
10. **AC10: Header cart button.**
    - `CartNavButton` (client) replaces the cart `Link` in `SiteNav`: a button with accessible name "Cart" ("Cart, N items" when N > 0).
    - It shows the `cart` icon plus a badge: JetBrains Mono 10px, `bg-horizon`, white, 16px circle, offset as in the design.
    - The badge shows `count` and is hidden at 0 and while loading.
    - Clicking toggles the drawer. On `/cart` itself, clicking does nothing extra; the page is already the cart.
11. **AC11: Cart drawer** (per design `.cart-drawer`):
    - **Container:** a `dialog` (`aria-modal`, labelled "Cart") fixed right, 420px wide (100% at ≤480px), sliding in over .25s, with an `rgba(18,33,42,.5)` overlay.
    - **Closing:** X ("Close cart"), overlay click, Esc, and route change. Body scroll is locked while open. Focus goes to the close button on open, is trapped inside, and returns to the opener on close.
    - **Contents, in order:**
      - Head "Your Cart (N)".
      - Free-shipping block (AC13).
      - Lines: 76×95 photo, name linking to the product, size label, qty stepper, line total, Remove.
      - Following any of its links closes the drawer, even when the link points at the current page.
      - Footer: Subtotal, View Cart (`ButtonCTA` → `CART_HREF`), disabled Checkout + note (AC15).
    - Empty → AC16.
12. **AC12: Cart page `/cart`.**
    - `app/[locale]/cart/page.tsx` is thin, dynamic, metadata title "Your Cart — WESTLINE" with `robots: noindex`, and renders `<CartPage/>`.
    - **Top:** breadcrumb Home / Cart. H1 "Your Cart" in Poppins 800 uppercase.
    - **Grid:** `1.7fr 1fr`, 48px gap, 1240px container; a single column at ≤900px.
    - **Left column:**
      - Free-shipping block.
      - One row per line: 110×138 photo on `bg-surface` (`#F9F9F9`, the design's surface-2), name link (16px/700), size label, line total (mono), qty stepper, Remove.
    - **Right column:** sticky (`top: 88px`; static at ≤900px, like the PDP buy column) summary "Order Summary":
      - Subtotal, "Shipping: Calculated at checkout", Total (= subtotal).
      - Disabled Checkout + note.
      - Continue Shopping → `PRODUCTS_HREF`.
      - "Taxes and shipping are calculated at checkout."
    - Loading shows a skeleton. A load error shows "Your cart couldn't be loaded right now." with a Retry button.
13. **AC13: Free shipping.**
    - `FREE_SHIPPING_THRESHOLD = 75` lives in `lib/shipping.ts`. The PDP trust text and the Shipping & Returns lines become functions of it, with the same rendered strings as today.
    - The page banner sits in a `bg-surface` box, as in the design.
    - Below the threshold: "Add $X more for free shipping." (`formatPrice`).
    - At or above: "You've unlocked free shipping!"
    - A progress bar (`bg-horizon` fill on `bg-border`; 5px on the page, 4px in the drawer) with `role="progressbar"` and `aria-valuenow`.
14. **AC14: Qty, remove, stock and price states.**
    - **Stepper:** buttons "Decrease quantity" / "Increase quantity" (`minus` / `plus` icons), with the qty in mono.
      - `−` at 1 is disabled; Remove is the explicit action.
      - `+` is disabled at `maxQty`.
    - Remove deletes the line.
    - Controls on a line are disabled while its mutation is pending.
    - **An `adjusted` line** shows "Only N left — quantity updated." (`text-danger`, mono 11.5px).
    - **A `soldOut` line** (Stock 0) shows "Sold out".
    - **An `unavailable` line** (the SizeKey no longer exists) shows "This size is no longer available."
    - **Both kinds of blocked line:**
      - The stepper is hidden, the total is struck through and excluded from the subtotal, and Remove stays.
      - Checkout stays blocked while any blocked line exists, and a cart-level note reads "Remove unavailable items to continue."
    - **`removedCount > 0`** shows "An item in your cart is no longer available and was removed." (covers deleted and unpublished products).
    - Prices are always the live Strapi price.
15. **AC15: Checkout.** In the drawer and on the page, Checkout is a disabled `ButtonCTA`, with the muted note "Checkout is coming soon." No checkout route is added.
16. **AC16: Empty state.** Drawer and page show "Your cart is empty." plus Continue Shopping → `PRODUCTS_HREF`. The page also hides the summary and the free-shipping block.
17. **AC17: Hidden, no dead UI:** promo code, recommended products, order notes, save-for-later / wishlist from cart, tax line, shipping estimate, PDP quantity picker.

## Technical Approach

**Strapi (`cms/`)**
- **Schemas:**
  - `src/components/cart/line.json`.
  - `src/api/cart/content-types/cart/schema.json`.
  - No SQL migration: new tables are auto-created and existing data is untouched.
- **`src/api/cart/routes/cart.ts`:** custom routes only, `config: { auth: false }`. `createCoreRouter` is not used, so there is no public CRUD.
- **`src/api/cart/controllers/cart.ts`:**
  - Resolves identity: a Bearer JWT is checked via `strapi.plugin('users-permissions').service('jwt').verify` and the user is loaded; otherwise `x-cart-token`.
  - Validates bodies and maps service errors to 400 / 401 / 404 / 409.
- **`src/api/cart/services/cart.ts`:** document-service reads and writes. It populates `Lines.Product` with `Images`, `BoardSizes` and `StandardSizes`.
- **`src/api/cart/cart-logic.ts`:** pure and unit-tested.
  - `sizeKeyOf(product, size)` and `findSize(product, key)`.
  - `addLine` / `setLineQty` / `removeLine` / `mergeLines` (sum + clamp).
  - `revalidate(lines)` → `{ lines, adjusted, soldOut, removedCount }`.
  - `toCartResponse`, which returns raw size fields so the frontend formats them.
- **Tests:** Vitest is added as a cms devDependency with an `npm test` script. Tests live in `cms/tests/cart/cart-logic.test.ts`.

**Frontend (`frontend/src/`)**
- **`lib/strapi/client.ts`:** `strapiRequest(path, { method, body, headers, label })`. It is `cache: 'no-store'`, never throws, and returns `{ ok, status, data }`. It stays the only Strapi caller.
- **`lib/strapi/cart.ts`:** `getCart`, `addCartLine`, `setCartLineQty`, `removeCartLine`, `mergeGuestCart(jwt, token)`, plus `StrapiCart` types and shape checks.
- **`lib/strapi/sizes.ts`:** `boardSizeKey`, `standardSizeKey`. These must match `cart-logic` and are covered by the same test cases on both sides.
- **`lib/cart/cookie.ts`:** `CART_COOKIE_NAME = 'westline_cart'`, `getCartCookieOptions()`, `readCartIdentity(request)` (session JWT + token).
- **`lib/cart/merge-on-auth.ts`:** `withGuestCartMerge(request, response, jwt)` calls `mergeGuestCart` and deletes the cookie. It catches and logs, so it never throws.
  - Called in `app/api/auth/{login,register,reset-password,oauth/complete}/route.ts` right after the session cookie is set.
  - It is one added line per route.
- **`lib/shipping.ts`:** `FREE_SHIPPING_THRESHOLD = 75`.
- **`app/api/cart/route.ts` (GET) and `app/api/cart/lines/route.ts` (POST / PATCH / DELETE):** thin. They validate the body, call `lib/strapi/cart.ts`, set the cookie on a new token, and return `toCartView(...)`.
- **`app/[locale]/cart/page.tsx`:** thin; `dynamic = 'force-dynamic'` plus metadata.
- **`components/cart/`** (app-level, appears once):
  - `cart.ts`: the `CartView` type, `toCartView`, `EMPTY_CART`.
  - `cart-texts.ts`.
  - `cart-context.ts`: `CartContext` and `useCart`.
  - `CartProvider.tsx` (`'use client'`):
    - State is the cart, status, `drawerOpen`, a pending-keys Set and the last error.
    - Mutations are `fetch` calls to `/api/cart/*`.
    - It also renders `<CartDrawer/>`, so the drawer exists on every page.
  - `CartNavButton.tsx`.
  - `CartDrawer.tsx`: dialog, focus trap, Esc, scroll lock. It reuses the `PhotoLightbox` effect pattern and closes on `usePathname()` change.
  - `CartLines.tsx`: one component with `variant: 'drawer' | 'page'`. Lines and the stepper are inlined in the map, with per-line pending lifted to the provider.
  - `ShippingProgress.tsx`.
  - `CartSummary.tsx`.
  - `CartPage.tsx`.
  - `index.ts`.
- **Detail pages:**
  - The `surfboard-detail.ts` and `standard-detail.ts` mappers add `documentId` and `sizes[].key`.
  - Both buy panels take a `documentId` prop and call `useCart().add`.
  - The add-error and "Adding…" strings go in `product-detail-texts.ts`.
  - `texts.trust.shipping` and `standard.shippingLines` are built from `FREE_SHIPPING_THRESHOLD`.
- **Icons:** add `plus` and `minus` to `SHAPES` in `Icon.tsx`.
- **Tests (TDD, Vitest + Testing Library):**
  - Mirrored under `src/__tests__/`.
  - `fetch` is mocked for the route handlers and `lib/strapi/cart.ts`.
  - The provider is wrapped in a test helper for components.

## Out of Scope

- Checkout, order content type, mock payment, order confirmation (cart-checkout epic). Checkout stays disabled.
- Promo codes, recommended products, order notes, save-for-later / wishlist from cart (customer-account-wishlist).
- Tax and shipping calculation.
- A PDP quantity picker. Add from catalog cards: the design has no add button on cards.
- **Stock:**
  - Reserving or decrementing stock on add (Seller Dashboard / checkout).
  - Concurrency guarantees beyond last write wins.
- **Tech debt, to be handled with checkout (cart-checkout epic):**
  - Abandoned guest carts are never cleaned up or expired; cart rows accumulate.
  - Cart creation (first `POST /lines` without a token) isn't rate-limited.
- Cross-tab live sync (a reload picks up changes).
- Any change to the Product schema.

## Standards Applied

- [frontend/components](../../../../../../standards/frontend/components.md):
  - one component and one return per file
  - `cart-texts.ts`
  - `app/` routing only
  - Strapi only through `lib/strapi/client.ts`, now with `strapiRequest`
  - URLs only via `lib/routes.ts`
  - mapper next to component
  - icons only via `Icon`
- [global/git-workflow](../../../../../../standards/global/git-workflow.md)

## Changelog

| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-10-10 | Ori Chai Matan | created | Initial shaping | — |
| 2026-10-10 | Ori Chai Matan | change | Built T1–T3. AC2: Strapi's DELETE is `DELETE /api/carts/current/lines/:productDocumentId/:sizeKey`, because koa-body doesn't parse DELETE bodies; the Next `DELETE /api/cart/lines` keeps its JSON body. Guest → user merge, when the user has no cart, re-owns the guest cart instead of copying it. Vitest added to `cms/` (`npm test`, `vitest.config.mts`, `tests/` excluded from the server build) | tasks.md T1–T3 |
| 2026-10-10 | Ori Chai Matan | change | Mappers: a missing `documentId` is treated like any other missing required field (`notFound()`); Strapi v5 always returns it. Size keys come from `boardSizeKey` / `standardSizeKey` in `lib/strapi/sizes.ts`; `boardSizeFromKey` labels an `unavailable` line. Add-to-cart state for both buy panels lives in one hook, `product-detail/shared/use-add-to-cart.ts` | tasks.md |
| 2026-10-10 | Ori Chai Matan | change | AC8: `CartProvider` takes `signedIn` from the locale layout (`getSession()`) and reloads the cart when it changes. Login and logout use `router.refresh()`, which keeps client state, so without this the header showed the previous session's cart (found in T21 E3/E5) | tasks.md |
| 2026-10-10 | Ori Chai Matan | change | AC11: `add()` records the focused element when it starts, because the Add to Cart button is disabled while pending and loses focus; focus now returns to it on close (found in T21 A2). The drawer's links close it on click, because Continue Shopping on `/products` doesn't change the pathname (found in T21 F5). The drawer mounts when opened and slides in (`animate-drawer-in`, `.25s`); it unmounts on close with no slide-out | tasks.md |
| 2026-10-10 | Ori Chai Matan | change | Design-fidelity choices: a `surface` colour token (`#F9F9F9`, the design's `--surface-2`) was added to `tailwind.config.ts` for the free-shipping banner and cart photo frames (`bg-background` is white, so neither showed). The summary is static at ≤900px. View Cart / Checkout are `ButtonCTA` stretched by a wrapper (`[&>*]:w-full`). Continue Shopping is a plain link styled as the design's outlined `.btn-cart-secondary`, because ButtonCTA has no outline variant. Summary amounts aren't monospace (as in the design); line totals are. Unit prices aren't shown, only line totals, as in the design | tasks.md |
| 2026-10-10 | Ori Chai Matan | change | Not covered by the spec: a failed quantity change or removal shows "Couldn't update your cart. Try again." (`role=alert`) and reloads the cart from the server. The header button is `aria-haspopup="dialog"` with `aria-expanded`. `CART_API_HREF` / `CART_LINES_API_HREF` were added to `lib/routes.ts`. `useCart` isn't exported from `components/cart/index.ts`: the layout imports that barrel and `cart-context.ts` calls `createContext`. The now-unused `cartLabel` was removed from `site-nav-texts.ts` | tasks.md |
| 2026-10-10 | Ori Chai Matan | change | Built T5–T19, verified T21, QA T22: PASS WITH NOTES (qa-report.md). Status → done | tasks.md |
| 2026-10-10 | Ori Chai Matan | fix | An expired or invalid `westline_session` made `/api/cart` and `/api/cart/lines` return 401, so Add to Cart failed. Cause: `jwtManagement: 'refresh'` issues 600-second access tokens (claims `type: access`, `exp − iat = 600`), login returns no refresh token, and the session cookie has no expiry, so it outlives its token 10 minutes after login. The rest of the app treats cookie presence as signed in (`getSession()`) and never checks the token. Fix: `respondAsShopper` in `lib/cart/route-helpers.ts` retries a 401 as a guest, clears the dead cookie and sets `x-westline-session: expired`; `CartProvider` calls `router.refresh()` on that header (router kept in a ref). The test that expected a 401 pass-through was replaced. 8 new tests (480 total). Live in Chrome with a garbage cookie and a real expired token: no 401, header flips to Sign In, Add to Cart works. The 10-minute sign-in lifetime itself belongs to account-signup-login (follow-up) | lib/cart/route-helpers.ts |
| 2026-10-10 | Ori Chai Matan | change | Note: the root cause of the expired-session 401s was fixed in account-signup-login (hotfix): 30-day JWTs (`legacy-support`), a 30-day `westline_session` cookie, and `getSession()` treating an expired token as signed out, so the layout's `signedIn` and the header agree with the token. The cart's guest fallback for a rejected session (`respondAsShopper`) stays as a safety net. Live: a signed-in Add to Cart writes to the user's cart and survives reload | account-signup-login |

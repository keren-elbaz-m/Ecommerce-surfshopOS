# Wishlist

> Epic: ../../epic.md
> Status: in-progress
> Branch: feat/customer-account-wishlist/wishlist
> Covers: cms/src/api/wishlist/**, cms/tests/wishlist/**, frontend/src/app/api/wishlist/**, frontend/src/components/wishlist/**, frontend/src/lib/strapi/wishlist.ts, frontend/src/lib/wishlist/**, frontend/src/__tests__/components/wishlist/**, frontend/src/__tests__/app/api/wishlist/**, frontend/src/__tests__/lib/strapi/wishlist.test.ts, frontend/src/__tests__/lib/wishlist/**

## Overview

Signed-in shoppers can save products from the catalog card heart and the product page, and see and remove them in My Account → Wishlist. Saves persist in Strapi per user. Guests who click a heart sign in and come back to the same page with the product saved. This completes the epic's "add/remove/view wishlist items" story. It wires the heart that product-card already exposes (AC8) and fills the Wishlist section that account-profile left as structure.

Design source: `westline-site-desing/` (28 Sep): `catalog.html` (`.prod-wish`), `profile.html` (`#panelWishlist`, slider, toast and remove scripts), `product-apparel-samurai-boardshort.html` (`.pdp-actions` / `.btn-wish`) and `product-surfboard-tideline.html` (`.pdp-actions-row` / `.btn-outline-sm`). Details are in `visuals/wishlist-notes.md`.

Design gaps (nothing invented; each was decided in shaping):
- The PDP controls have no saved state. They follow the card convention.
- No order is shown. Newest first.
- No failure state. The UI reverts quietly.
- No unavailable-product state. Unpublished or deleted products drop out; sold-out products stay as normal cards.
- No toggle feedback outside the account page.

## Goals & User Stories

- As a signed-in shopper, I can save a product from its catalog card or product page, and unsave it the same way.
- As a signed-in shopper, I can see everything I've saved in My Account and remove items there.
- As a guest, clicking a heart lets me sign in and returns me to the same page with the product saved.

## Acceptance Criteria

1. AC1: On the catalog, each card shows the heart (design `.prod-wish`). For a signed-in shopper it reflects the saved state (`aria-pressed`, filled, label "Remove from wishlist" / "Add to wishlist"). Clicking it toggles save/remove immediately (optimistic). The state persists across reloads.
2. AC2: Standard product page: a `.btn-wish` (52px, bordered, 20px ink heart, label "Add to wishlist") sits beside Add to Cart in `.pdp-actions`. Surfboard product page: a `.btn-outline-sm` "Wishlist" button with a 15px heart sits in `.pdp-actions-row`, filling the row since Share is out. Both toggle like AC1. When saved: `aria-pressed="true"`, label "Remove from wishlist", heart filled (design gap: card convention).
3. AC3: A guest's heart (card or product page) navigates to `/auth/login?next=<current path and query, plus wishlist=<slug>>`. After signing in with a password, or registering via the login page's link, the shopper lands on that same page with the product saved and the heart filled, and `wishlist=` is removed from the URL without a new history entry.
4. AC4: The automatic save in AC3 happens only when the same tab set a one-time marker for that slug at click time, within the last 10 minutes. Opening a URL that carries `?wishlist=<slug>` without the marker saves nothing (the param is still stripped). The marker is cleared after use. A slug not matching `^[a-z0-9]+(?:-[a-z0-9]+)*$` (max 120) is never sent.
5. AC5: My Account → My Account tab → Wishlist shows the saved products, newest first, as the shared product cards (heart pressed, label "Remove from wishlist"), each linking to its product, with the count "N saved" ("1 saved" too). The list is rendered server-side with the page.
6. AC6: Wishlist slider (design `profile.html`): 240px cards, 20px gap, whole cards only. Prev/next arrows ("Scroll wishlist left/right") scroll one viewport and hide at either end or when nothing overflows. At ≤640px the arrows are hidden and each card is full-width (swipe).
7. AC7: Removing from the account section fades the card out (opacity 0, scale .96, 250ms), shows the toast "Removed from wishlist." for 2.2s, and updates the count. Removing the last card shows the existing empty state ("Your wishlist is empty. Browse Surfboards or Apparel to add favorites.").
8. AC8: If a save or remove fails, the heart (or the account card) returns to its previous state with no message. A 401 from an expired session behaves like a guest click (AC3) for a save, and goes to sign-in returning to the same page for a remove.
9. AC9: Saving an already-saved product keeps one entry. Removing a product that isn't saved succeeds as a no-op. An unknown slug returns 404 and changes nothing.
10. AC10: Products that are deleted no longer appear in the wishlist (their rows are cleaned up). Sold-out products still appear as normal cards. *(Products have no draft/publish — `draftAndPublish: false` — so there is no "unpublished" state; amended 2026-10-14.)*
11. AC11: Wishlist data is only readable and writable for the session's own user. Every wishlist route returns 401 without a valid session, no request can name another user, and `wishlist-item` has no core CRUD route.
12. AC12: Card and slider layout matches the design at 1280, 900 and 375px with no page-level horizontal scroll.

## Technical Approach

**CMS: `cms/src/api/wishlist/`** (custom routes only, like `api/account` and `api/cart`):
- Content type `wishlist-item` (`api::wishlist.wishlist-item`, `draftAndPublish: false`):
  - `User` manyToOne to `plugin::users-permissions.user`.
  - `Product` manyToOne to `api::product.product`.
  - Timestamps provide `createdAt` for ordering.
- `wishlist-logic.ts`: `parseSlug` (the format check above) and a `WishlistError` (`bad-request | unauthorized | not-found`).
- `wishlist-store.ts`:
  - `list(userDocumentId)`: the user's items with their products, `createdAt desc`, returning card fields (`Name`, `Slug`, `Price`, `Images`, `Category.Slug`).
  - `add(userId, slug)`: resolves the published product by slug (404 if none) and creates the item only if the (user, product) pair doesn't exist.
  - `remove(userId, slug)`: deletes the pair if present.
  - Rows whose product was deleted are skipped in `list` and cleaned up opportunistically.
- `routes/wishlist.ts`, all `config: { auth: false }`:
  - `GET /wishlist` → `wishlist.list`
  - `PUT /wishlist/items/:slug` → `wishlist.add`
  - `DELETE /wishlist/items/:slug` → `wishlist.remove`
  - (path params, since koa-body doesn't parse DELETE bodies)
- `controllers/wishlist.ts`: `requireUserId` via `createIdentify` (`api/cart/identity.ts`), the same pattern as `api/account/controllers/account.ts`.

**Frontend:**
- `lib/wishlist/slug.ts`: the shared slug check. `lib/wishlist/pending.ts`: the sessionStorage marker (`markPending(slug)`, `takePending(slug)`), try/catch, 10-minute TTL.
- `lib/strapi/wishlist.ts`: `getWishlist(jwt)` → `ProductCard[]` via `toProductCard`; `addToWishlist(jwt, slug)`; `removeFromWishlist(jwt, slug)`. Results are typed `ok | unauthorized | not-found | failed`, built on `strapiRequest`.
- `app/api/wishlist/route.ts` (GET → `{ slugs, items }`) and `app/api/wishlist/items/[slug]/route.ts` (PUT, DELETE). They are cookie-gated (401), validate the slug (400), and return no-store responses, the same shape as `app/api/account/profile/route.ts`.
- `lib/routes.ts`: add `WISHLIST_API_HREF` and `wishlistItemApiHref(slug)`.
- `components/wishlist/` (app-level, per the components standard):
  - `WishlistProvider.tsx` + `wishlist-context.ts` + `use-wishlist.ts`:
    - The saved-slug set, loaded from `/api/wishlist` when `signedIn` (or seeded from `initialSlugs`).
    - `toggle(slug)`: optimistic, reverted on failure; a guest or a 401 goes through `markPending` + `router.push(loginHref(withWishlistParam(currentUrl, slug)))`.
    - A landing effect reads `?wishlist=`, saves only if `takePending(slug)`, then `router.replace` without the param.
    - Server-safe barrel split like `components/cart`.
  - `WishlistProductCard.tsx`: a client wrapper passing `isFavorite` / `onToggleFavorite` from context to the shared `ProductCard`, which is unchanged.
  - `WishlistButton.tsx`: the PDP control, with `variant: 'icon' | 'labelled'` (Samurai `.btn-wish` / Tideline `.btn-outline-sm`). A single return that chooses between the variants.
  - `WishlistSection.tsx`: the account section (count, slider with arrows, remove fade, toast, empty state). The removal toast is local, since no shared toast exists.
  - `wishlist-texts.ts`, `*.styles.ts`, `index.ts`.
- Small edits in other specs' files (Changelog rows on each owner when made):
  - `app/[locale]/layout.tsx`: `WishlistProvider signedIn={…}` beside `CartProvider`.
  - `components/catalog/CatalogPage.tsx`: `WishlistProductCard` instead of `ProductCard`.
  - `components/product-detail/standard/StandardBuyPanel.tsx`: `.pdp-actions` + `WishlistButton variant="icon"`.
  - `components/product-detail/surfboard/SurfboardBuyPanel.tsx`: `.pdp-actions-row` + `WishlistButton variant="labelled"`.
  - `components/account/AccountOverview.tsx`: renders `WishlistSection` instead of the static wishlist half.
  - `app/[locale]/account/page.tsx`: `getWishlist(jwt)` server-side, passed down, seeding the provider.
  - `lib/routes.ts`: the two additions above.

## Out of Scope

- Add-to-cart from the wishlist.
- Share.
- A count badge on the nav heart.
- A guest or localStorage wishlist.
- Google sign-in in this spec's ACs and QA (V1 exclusion; the existing `next` carry is untested here).
- Home "Featured gear" and the PDP "You might also like" rail: not built, so there's nothing to wire.
- Toggle feedback other than the heart.
- A wishlist size cap.
- Changing the shared `ProductCard`.

## Standards Applied

- [frontend/components](../../../../../../standards/frontend/components.md): one component and one return per file, texts files, `app/` as routing only, Strapi via `lib/strapi/client.ts`, URLs via `lib/routes.ts`, server-safe barrels.
- [global/git-workflow](../../../../../../standards/global/git-workflow.md): branch `feat/customer-account-wishlist/wishlist`, stacked on `account-profile`.
- [global/tech-stack](../../../../../../standards/global/tech-stack.md): no new dependencies.

## Changelog

| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-10-14 | Ori Chai Matan | created | Initial shaping (tasks.md not yet created, by request) | — |
| 2026-10-14 | Ori Chai Matan | change | tasks.md created (T1-T24) and T1-T23 implemented on `feat/customer-account-wishlist/wishlist` (commit `798d733`, stacked on account-profile). Deviations from the Technical Approach: `useWishlist` lives in `wishlist-context.ts` (as `useCart` does in `cart-context.ts`), so there is no separate `use-wishlist.ts`; added `use-wishlist-slider.ts` (`fitSlider` = the design's slider math), `wishlist.types.ts` and two styles files; the store keys on the user's `documentId` (document service, like cart); `strapiRequest` gained `PUT`; on the account page the "Removed from wishlist." toast appears once the 250ms fade ends and the DELETE succeeded (not at click), so a failed removal shows nothing (AC8); a Strapi failure on the account page renders the empty wishlist. AC10 amended: products have no draft/publish state | tasks.md |
| 2026-10-14 | Ori Chai Matan | change | T24 QA: PASS WITH NOTES, 12/12 AC (live curl + headless Chromium, three throwaway users deleted afterwards) | qa-report.md |

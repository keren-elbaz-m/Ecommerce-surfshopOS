# Tasks — Wishlist

> Epic: customer-account-wishlist
> Spec: 2026-10-14-wishlist
> Branch: feat/customer-account-wishlist/wishlist

## Backend — wishlist API (cms)

- [x] T1 Add the `wishlist-item` content type (`cms/src/api/wishlist/content-types/wishlist-item/schema.json`): `User` manyToOne users-permissions user, `Product` manyToOne product, `draftAndPublish: false`
- [x] T2 Write failing tests for `cms/src/api/wishlist/wishlist-logic.ts` (`parseSlug`, `WishlistError`) and `wishlist-store.ts` (list newest first with card fields, deleted products skipped; idempotent add; unknown slug → not-found; idempotent remove; always scoped to the given user) (AC9-AC11)
- [x] T3 Implement `wishlist-logic.ts` + `wishlist-store.ts` + `services/wishlist.ts` to pass T2
- [x] T4 Write failing tests for `controllers/wishlist.ts` + `routes/wishlist.ts`: exactly `GET /wishlist`, `PUT /wishlist/items/:slug` and `DELETE /wishlist/items/:slug`, all `auth: false`; 401 for guests or a bad JWT, 400 for a bad slug, 404 for an unknown product, 200 otherwise (AC9, AC11)
- [x] T5 Implement the controller and routes to pass T4
- [x] T6 Manual curl check against local Strapi with two throwaway users (deleted afterwards): 401 without a session; add twice → one item; remove twice → no-op; unknown slug → 404; B can't see A's list; core `/api/wishlist-items` is not routed (AC9, AC11) **Verified 2026-10-14**: guest/garbage JWT → 401; add ×2 → one item; list newest first; B's list empty and B's DELETE of A's product leaves A's list intact; remove ×2 → no-op; unknown slug → 404; `Bad%20Slug` → 400; core `/api/wishlist-items` → 404.

## Frontend — data + routes

- [x] T7 Write failing tests for `lib/wishlist/slug.ts` (`isWishlistSlug`) and `lib/wishlist/pending.ts` (`markPending` / `takePending`: one-time use, same slug only, 10-minute TTL, storage errors fail closed) (AC4)
- [x] T8 Implement `lib/wishlist/slug.ts` + `pending.ts` to pass T7
- [x] T9 Write failing tests for `lib/strapi/wishlist.ts` (`getWishlist` → `ProductCard[]` via `toProductCard`; `addToWishlist` / `removeFromWishlist` → `ok | unauthorized | not-found | failed`) (AC5, AC9)
- [x] T10 Implement `lib/strapi/wishlist.ts` to pass T9
- [x] T11 Add `WISHLIST_API_HREF` + `wishlistItemApiHref(slug)` to `lib/routes.ts` (site-nav Changelog row)
- [x] T12 Write failing tests for `app/api/wishlist/route.ts` (GET) and `app/api/wishlist/items/[slug]/route.ts` (PUT, DELETE): 401 without a cookie, 400 for a bad slug, 404/401/502 mapping, no-store (AC9, AC11)
- [x] T13 Implement both route handlers to pass T12

## Frontend — provider + hearts

- [x] T14 Write failing tests for `components/wishlist/WishlistProvider.tsx` / `use-wishlist`: seeded or loaded slugs; optimistic toggle; quiet revert on failure; a guest or 401 save → `markPending` + login with `next` + `wishlist=`; a 401 remove → login with `next`; the landing effect auto-saves only with a matching marker, then strips the param via `router.replace` (AC1, AC3, AC4, AC8)
- [x] T15 Implement `WishlistProvider`, `wishlist-context.ts`, `use-wishlist.ts`, `wishlist-texts.ts` and the server-safe `index.ts` to pass T14
- [x] T16 Write failing tests for `WishlistProductCard.tsx` (feeds `isFavorite` / `onToggleFavorite` into the shared card) and `WishlistButton.tsx` (`icon` = Samurai `.btn-wish`, `labelled` = Tideline `.btn-outline-sm` "Wishlist"; `aria-pressed`, label swap, filled when saved) (AC1, AC2)
- [x] T17 Implement `WishlistProductCard` + `WishlistButton` to pass T16
- [x] T18 Wire the layout (`WishlistProvider signedIn`), `CatalogPage` (`WishlistProductCard`), the PDP mappers (`slug`) and buy panels (`.pdp-actions` / `.pdp-actions-row` + `WishlistButton`), and update the affected existing tests. Changelog rows on account-signup-login, catalog-listing-page, standard-product-detail-page and surfboard-detail-page `strapiRequest` also needed `PUT` in its method type (site-nav Changelog row). The superseded hidden-heart assertions in `CatalogPage.test`, `ProductDetailPage.test` and `SurfboardBuyPanel.test` now assert the heart/button is present.

## Frontend — account section

- [x] T19 Write failing tests for `WishlistSection.tsx`: "N saved"; cards newest first; arrows hidden at the ends; remove → 250ms fade, toast "Removed from wishlist." for 2.2s, count update; last removal → empty state; failed remove restores the card (AC5-AC8)
- [x] T20 Implement `WishlistSection` (+ styles) to pass T19
- [x] T21 Wire `AccountOverview` + `app/[locale]/account/page.tsx` (server `getWishlist(jwt)`, seed the provider); update account-profile's tests (account-profile Changelog row) (AC5)

## Verification

- [x] T22 Full verify: frontend vitest, tsc, lint and build (with the dev server stopped); cms tsc and tests (testing skill) **2026-10-14** (dev server stopped for the build): frontend 92 files / 823 tests, `tsc` clean, lint clean (pre-existing font warning only), build passes (`/api/wishlist`, `/api/wishlist/items/[slug]` dynamic); cms `tsc` clean, 20 files / 292 tests.
- [x] T23 Live checks: the curl matrix (T6); headless Chromium — guest heart on a catalog page with a query → login (password, and via the register link) → same URL, heart filled, param gone; a `?wishlist=` link without the marker saves nothing; PDP buttons on both product types; account slider, remove fade and toast, empty state; 1280/900/375. Throwaway users deleted afterwards **Done 2026-10-14** — see qa-report.md (three throwaway users, deleted afterwards with their wishlist item).
- [x] T24 Run QA against AC1-AC12 (qa skill) → qa-report.md **Run 2026-10-14:** PASS WITH NOTES, 12/12 AC — see qa-report.md

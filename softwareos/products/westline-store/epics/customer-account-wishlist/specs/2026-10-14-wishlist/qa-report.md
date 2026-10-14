# QA Report — Wishlist

> Spec: spec.md
> Date: 2026-10-14 · QA by: Ori Chai Matan · Mode: setup
> Verdict: PASS WITH NOTES

## Setup

- **Branch:** `feat/customer-account-wishlist/wishlist` at `74541a5` (wishlist commit `798d733`), stacked on account-profile.
- **Servers:** cms on :1337 (restarted after the new content type), frontend `next dev` on :3000.
- **Browser runs:** headless Chromium (Playwright).
- **Test data:** three throwaway `qa-acct-*@example.com` users (A and B for the curl matrix, C registered during the run), deleted afterwards (`DELETED {"count":3}`, wishlist rows `{"count":1}`).

## Acceptance Criteria

| AC | Criterion | Result | Evidence |
|---|---|---|---|
| AC1 | Catalog card heart: saved state, optimistic toggle, persists | pass | **Runtime:** a heart on `/products?page=2` pressed after sign-in; unsave → `aria-pressed=false` and gone from `/api/wishlist`; save + reload → `true`. **Tests:** `WishlistProductCard.test.tsx`, `WishlistProvider.test.tsx` (optimistic save), `CatalogPage.test.tsx` (16 hearts) |
| AC2 | PDP controls (`.btn-wish` / labelled `.btn-outline-sm`), saved state | pass | **Runtime, Samurai:** a 52px icon button beside Add to Cart, `aria-pressed` true after click. **Runtime, Tideline:** "Wishlist" in JetBrains Mono, full row width, label → "Remove from wishlist", heart fill `rgb(16,24,40)`. Screenshots checked. **Tests:** `WishlistButton.test.tsx`, `StandardBuyPanel.test.tsx`, `SurfboardBuyPanel.test.tsx` |
| AC3 | Guest heart → login (password / register link) → same page, saved, param stripped | pass | **Runtime, password:** catalog heart → `/auth/login?next=/products?page=2&wishlist=big-happy` → landed on `/products?page=2`, heart pressed, saved server-side. **Runtime, register:** PDP heart → login → register link (`next` carried) → registered C → `/products/tidal-rash-guard`, pressed, saved. **Test:** `WishlistProvider.test.tsx` (landing) |
| AC4 | Auto-save only with the one-time same-tab marker; slug format | pass | **Runtime:** the marker is set at click (`{"slug":"big-happy","at":…}`); `/products?wishlist=tidal-rash-guard` without the marker → param stripped, nothing saved. **Tests:** `pending.test.ts` (one-time, same slug, 10-minute TTL, fails closed), `slug.test.ts`, provider (no marker / other slug / malformed slug / still signed out) |
| AC5 | Account section: saved cards newest first, "N saved", server-rendered | pass | **Runtime:** 7 cards in the same order as `/api/wishlist` (newest first), "7 saved", each linking to its product. **Tests:** `WishlistSection.test.tsx`, `app/[locale]/account/page.test.tsx` (server fetch, empty on failure) |
| AC6 | Slider: 240px whole cards, arrows hide at the ends, full-width on mobile | pass | **Runtime:** at 1280, card 240 and grid 1020 (4 cards), arrows `[hidden, visible]` → after Next scrollLeft 780 and `[visible, hidden]`; at 900, grid 500 (2 cards); at 375, card 343 = grid and arrows `display:none`. **Test:** `use-wishlist-slider.test.tsx` (`fitSlider`) |
| AC7 | Remove: 250ms fade, toast 2.2s, count update, empty state | pass | **Runtime:** mid-fade opacity 0.39 and scale 0.975; then 7 → 6 cards, "6 saved", toast "Removed from wishlist." (opacity ~1), toast opacity 0 after 2.6s; removing all → "0 saved" + the empty-state copy; reload → 0. **Test:** `WishlistSection.test.tsx` |
| AC8 | Failures revert quietly; 401 behaves like a guest | pass | **Runtime:** a forced 502 on a catalog heart → stays unpressed, no navigation; a forced 502 on an account removal → the card stays (opacity 1), no toast. **Tests:** provider (502 / offline revert, 401 save → login with marker, 401 remove → login without), `WishlistSection.test.tsx` |
| AC9 | Idempotent save and remove; unknown slug 404 | pass | **Runtime (curl):** add ×2 → one entry; remove ×2 → 200 and no-op; `no-such-product-xyz` → 404. **Tests:** `cms/tests/wishlist/wishlist-store.test.ts`, `wishlist-controller.test.ts`, `__tests__/app/api/wishlist/route.test.ts` |
| AC10 | Deleted products drop out; sold-out stays | pass | **Test:** `wishlist-store.test.ts` (a null product is dropped and its row deleted; duplicates collapsed). Sold-out products are ordinary cards (no sold-out filter in the store). Products have no draft/publish state (AC10 amended) |
| AC11 | Owner-only, 401 without a session, no core CRUD | pass | **Runtime (curl):** guest and garbage JWT → 401; B's list `[]`; B's DELETE of A's product leaves A's list intact; `/api/wishlist-items` → 404. **Tests:** controller (401 for guest, none and bad JWT on all three routes; routes table), route handlers (401 without cookie) |
| AC12 | Layout at 1280/900/375, no horizontal scroll | pass | **Runtime:** `scrollWidth > innerWidth` false at all three widths on `/account#wishlist`; screenshots of the account section (1280, 375) and both PDPs checked against the design |

## Checks Run

- tests (frontend): `npx vitest run` — pass (92 files, 823 tests)
- typecheck (frontend): `npx tsc --noEmit` — pass
- lint (frontend): `npm run lint` — pass (1 pre-existing warning, layout custom font)
- build (frontend): `npm run build` — pass (`/api/wishlist`, `/api/wishlist/items/[slug]` dynamic). Run with the dev server stopped
- typecheck (cms): `npx tsc --noEmit` — pass
- tests (cms): `npm test` — pass (20 files, 292 tests)

## Issues Found

(none)

## Notes

- **Hygiene:** no PR yet. One PR will carry account-profile + wishlist + the home hero fix; `> PR:` is recorded then. T1–T23 `[x]`; T24 is closed by this run.
- **Superseded ACs in other specs** (Changelog rows added on each): catalog-listing-page AC12 (no heart), surfboard-detail-page AC12 (wishlist not built), standard-product-detail-page AC16 (wishlist hidden).
- **Deviations** are recorded in the spec Changelog:
  - The toast appears after the fade, once the DELETE succeeded.
  - `useWishlist` lives in `wishlist-context.ts`.
  - `strapiRequest` gained `PUT`.
- **Pre-existing test stderr** (not from this spec): `SiteNav.test.tsx` act warnings and the `CartDrawer.test.tsx` jsdom navigation notice.

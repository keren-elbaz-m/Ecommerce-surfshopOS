# QA Report — Order History

> Spec: spec.md
> Date: 2026-10-14 · QA by: Ori Chai Matan · Mode: setup
> Verdict: PASS WITH NOTES

## Setup

- **Branch:** `feat/customer-account-wishlist/order-history` at `d3098ca` (stacked on the wishlist branch).
- **Servers:** cms on :1337, frontend `next dev` on :3000. Browser runs in headless Chromium.
- **Test data:** two throwaway `qa-acct-*@example.com` users placed three real orders through the cart and order API (demo card 4242…). A got two orders a second apart and B got one. Statuses were then set to shipped / cancelled (A) and delivered (B).
- **Cleanup:** the orders were deleted with stock restored per line (`S 6→8`, `M 15→16`, `S 14→16`), along with 2 carts and the 2 users.

## Acceptance Criteria

| AC | Criterion | Result | Evidence |
|---|---|---|---|
| AC1 | Every own order, newest first, server-rendered; empty state when none | pass | **Runtime:** A sees `WL-180003` then `WL-778967` (placed 1s apart, newest first); B sees only `WL-503807`. **Tests:** `order-store.test.ts` (`listForUser`: own filter, `PlacedAt:desc`), `OrderHistorySection.test.tsx` (empty state), `app/[locale]/account/page.test.tsx` |
| AC2 | Summary: preview, number, "Placed …", "N items", pill, total, chevron | pass | **Runtime:** "WL-180003 Placed Oct 14, 2026 3 items SHIPPED $182" (Qty 2 + 1 = 3 items), with a preview image; "N items" is `display:none` at 375 and shown at 900. **Tests:** `order-history.test.ts`, `OrderHistorySection.test.tsx` |
| AC3 | Pills: Processing / Shipped / Delivered / Cancelled | pass | **Runtime:** Shipped `rgb(21,94,239)` on `rgba(21,94,239,.12)`; Cancelled `rgb(102,112,133)` on `rgb(249,249,249)` (the Processing style, design gap); B shows DELIVERED. **Test:** `order-history.test.ts` (all four, plus unknown → neutral) |
| AC4 | Toggle with `aria-expanded`/`aria-controls`, chevron, height transition, several open | pass | **Runtime:** closed height 0 / `visibility:hidden` → open height 268, chevron rotated 180°, then collapses back to 0. **Test:** two orders open at once, aria attributes |
| AC5 | Lines (thumb or initials, "{size} · Qty N", line total), Subtotal / Shipping / Total; no Track link | pass | **Runtime:** "Tidal Rash Guard S · Qty 2 $98 · Boardwalk Hybrid Shorts M · Qty 1 $59 · Subtotal $157 · Shipping $25 · Total $182"; image thumbs; 0 "Track shipment" links. **Tests:** initials ("Surf Wax — 3 Pack" → "SW"), "Free" at 0 |
| AC6 | Demo note under a non-empty list | pass | **Runtime:** visible for A. **Test:** absent when empty |
| AC7 | Owner-only `GET /orders`, 401 without a session, no internal fields | pass | **Runtime (API):** guest → 401; A and B each see only their own; response keys are the owner view only (no ids, idempotency key or payment reference). **Tests:** `order-controller.test.ts` (401 for none / guest / bad JWT; routes table), `order-store.test.ts` (no internal fields) |
| AC8 | Strapi failure → page renders with the empty state | pass | **Tests:** `getOrders` returns null on 401 / 500 / bad body / network; the page passes `orders ?? []` (`page.test.tsx`) |
| AC9 | Layout 1280 / 900 / 375, no horizontal scroll, the summary wraps | pass | **Runtime:** `scrollWidth > innerWidth` false at 900 and 375; at 375 the pill and total wrap to a second row (design `flex-wrap`). Screenshots checked |

## Checks Run

- tests (frontend): `npx vitest run` — pass (94 files, 852 tests)
- typecheck (frontend): `npx tsc --noEmit` — pass
- lint (frontend): `npm run lint` — pass (1 pre-existing font warning)
- build (frontend): `npm run build` — pass (dev server stopped for it)
- typecheck (cms): `npx tsc --noEmit` — pass
- tests (cms): `npm test` — pass (20 files, 298 tests)

## Issues Found

(none)

## Notes

- **Hygiene:** no PR yet. This is a follow-up PR stacked on the account-profile + wishlist PR.
- **Edits in checkout's files** (`cms/src/api/order/**`, `cms/tests/order/**`, `lib/strapi/orders.ts`, `orders.test.ts`) are recorded in checkout's Changelog.
- **Pre-existing test stderr:** `SiteNav.test.tsx` act warnings and the `CartDrawer.test.tsx` jsdom navigation notice.

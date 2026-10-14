# Order History

> Epic: ../../epic.md
> Status: in-progress
> Branch: feat/customer-account-wishlist/order-history
> Covers: frontend/src/components/order-history/**, frontend/src/__tests__/components/order-history/**

## Overview

A signed-in shopper's past orders in My Account → Order History, replacing account-profile's empty placeholder. Each order is an expandable card (number, date, item count, status, total) that opens to its lines and totals, all from the order's snapshot. Data comes from a new owner-only `GET /orders` on the existing order API (owned by cart-checkout/checkout, which gets a recorded edit). Design: `westline-site-desing/profile.html` `#panelOrders` (see visuals/order-history-notes.md). This is the tech plan's `order-history` candidate. Guest-order linking is moot because checkout requires sign-in.

Design gaps (decided): the Cancelled pill (styled like Processing), item initials when there's no image, date format (en-US, UTC), "N items" = total quantity. Omitted: "Track shipment" (live tracking is a V1 exclusion).

## Goals & User Stories

- As a signed-in shopper, I can see every order I've placed, newest first, with its status and total.
- As a signed-in shopper, I can open an order to see what I bought and what I paid.

## Acceptance Criteria

1. AC1: My Account → My Account tab → Order History lists every order of the signed-in shopper, newest first (by placed date), rendered with the page. With none, the existing empty state "You haven't placed any orders yet." shows.
2. AC2: Each order is a card whose summary button shows: a 52px preview of the first line's image, the order number (e.g. WL-104233), "Placed Aug 22, 2026", "N items" (sum of quantities; "1 item" for one; hidden at ≤600px), a status pill and the total, plus a chevron.
3. AC3: Status pills: Processing (grey), Shipped (blue tint), Delivered (green tint), Cancelled (grey; design gap). Labels are capitalised.
4. AC4: Clicking a summary toggles its detail open/closed (several can be open), with `aria-expanded` / `aria-controls`, the chevron rotating and a height transition (none with reduced motion).
5. AC5: The detail lists each line: a 44px thumbnail (or initials when there's no image), name, "{size} · Qty N", line total. Then Subtotal, Shipping ("Free" when 0) and Total. No "Track shipment" link.
6. AC6: Under a non-empty list: "Order status shown here is for demo purposes and does not reflect live carrier tracking."
7. AC7: Orders come only from the session's own user: `GET /orders` answers 401 without a valid session, never returns another user's orders, and exposes no ids, payment reference or idempotency key (same owner view as the confirmation page).
8. AC8: If Strapi fails to return orders, the page still renders, with the empty state.
9. AC9: Layout matches the design at 1280/900/375 with no page-level horizontal scroll; the summary wraps on narrow screens.

## Technical Approach

**CMS (checkout's `cms/src/api/order/`, small recorded edit):**
- `order-store.ts`: add `listForUser(user)` → `findMany({ filters: { User: { id: user.userId } }, sort: 'PlacedAt:desc', populate: ORDER_POPULATE })` mapped with the existing `toOrderResponse` (the owner view).
- `routes/order.ts`: add `{ GET /orders → order.list, auth: false }`.
- `controllers/order.ts`: `list` uses the existing `requireUser` and returns `{ orders }`.
- Tests: `cms/tests/order/order-store.test.ts` and `order-controller.test.ts`.

**Frontend:**
- `lib/strapi/orders.ts` (checkout's): `getOrders(jwt): Promise<StrapiOrder[] | null>`, reusing `isOrder`, `bearer` and `LABEL`; invalid entries are dropped.
- `components/order-history/` (app-level):
  - `order-history.ts`: `toOrderHistoryItem(order)` → `{ orderNumber, placedLabel, itemCount, status: { key, label }, total, preview, lines[{ key, name, spec, lineTotal, image | initials }], subtotal, shipping, totalRow }`. It uses `formatPrice` (product-card), `strapiMediaUrl` and Intl `en-US` with UTC.
  - `OrderHistorySection.tsx` (client): section head and empty state (strings moved from account-texts), the list with open state as a `Set` in the parent (components standard: lift per-item state), and the note.
  - `order-history-texts.ts`, `order-history.styles.ts`, `index.ts`.
- `components/account/AccountOverview.tsx` (account-profile): renders `OrderHistorySection id={SECTION_IDS.orders}` instead of the static half.
- `app/[locale]/account/page.tsx`: `Promise.all([getWishlist, getOrders])`, passing `orders ?? []` down. `AccountPage` gets an `orders` prop.
- No new route handler: the list is server-rendered only.

## Out of Scope

- Live tracking and "Track shipment".
- Order detail pages, reorder and cancel.
- Pagination.
- Linking guest orders.
- Admin status changes (already in checkout's admin).

## Standards Applied

- frontend/components, global/git-workflow, global/tech-stack.

## Changelog

| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-10-14 | Ori Chai Matan | created | Initial shaping | — |
| 2026-10-14 | Ori Chai Matan | change | Implemented T1-T12 on `feat/customer-account-wishlist/order-history` (commit `d3098ca`). `OrderHistorySection` takes the raw `StrapiOrder[]` and maps with `toOrderHistoryItem`; the detail animates via grid rows with an inline `visibility` (closed panels leave the tab order); the initials regex uses `new RegExp` (tsconfig target). T13 QA: PASS WITH NOTES, 9/9 AC | qa-report.md |

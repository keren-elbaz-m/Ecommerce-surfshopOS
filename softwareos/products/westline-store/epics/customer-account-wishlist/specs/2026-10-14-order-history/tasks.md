# Tasks — Order History

> Epic: customer-account-wishlist
> Spec: 2026-10-14-order-history
> Branch: feat/customer-account-wishlist/order-history

## Backend (cms, checkout's order API)
- [x] T1 Failing tests: `listForUser` (own user filter, PlacedAt desc, owner view) in order-store.test.ts; `list` controller (401 guest/bad JWT, `{ orders }`) and the routes table in order-controller.test.ts (AC1, AC7)
- [x] T2 Implement `listForUser`, the `order.list` controller and `GET /orders` to pass T1
- [x] T3 Live curl with two throwaway users (place an order each via the checkout API): 401 without session; each sees only their own; no internal fields (AC7) **Verified 2026-10-14** — guest 401; A sees its 2 orders newest first, B its 1; owner-view keys only (see qa-report.md).

## Frontend
- [x] T4 Failing tests: `getOrders` in orders.test.ts (200 / 401 / bad shape / network) (AC7, AC8)
- [x] T5 Implement `getOrders`
- [x] T6 Failing tests: `toOrderHistoryItem` (date UTC, item count, status labels incl. cancelled, Qty spec, Free shipping, initials, preview) (AC2, AC3, AC5)
- [x] T7 Implement order-history.ts + texts
- [x] T8 Failing tests: OrderHistorySection (empty state, newest-first order, toggle with aria + several open, detail content, note only when non-empty, no Track link) (AC1-AC6)
- [x] T9 Implement OrderHistorySection + styles + barrel
- [x] T10 Wire AccountOverview / AccountPage / account page.tsx (Promise.all, `?? []`); update account tests (AC1, AC8); Changelog rows on account-profile and checkout

## Verification
- [x] T11 Full verify (dev server stopped for the build): frontend vitest/tsc/lint/build, cms tsc/test **2026-10-14:** frontend 94 files / 852 tests, tsc, lint, build pass; cms tsc, 20 files / 298 tests.
- [x] T12 Live browser check: place an order as a throwaway user, open My Account, expand/collapse, 1280/900/375, then delete the users **Done 2026-10-14** — see qa-report.md (orders deleted with stock restored; users and carts deleted).
- [x] T13 QA against AC1-AC9 (qa skill) → qa-report.md **Run 2026-10-14:** PASS WITH NOTES, 9/9 AC — see qa-report.md.

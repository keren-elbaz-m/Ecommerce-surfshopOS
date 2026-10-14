# References for Order History

## Related Specs

- **[Related]** [cart-checkout / 2026-10-11-checkout](../../../cart-checkout/specs/2026-10-11-checkout/spec.md): owns `cms/src/api/order/**`, `cms/tests/order/**` and `frontend/src/lib/strapi/orders.ts`. This spec adds `listForUser` + `GET /orders` + `getOrders` there and reuses `toOrderResponse` (the owner view), `StrapiOrder` and `isOrder`. Kept separate because the order *history* view belongs to this epic; the edits are additive.
- **[Related]** [2026-10-13-account-profile](../2026-10-13-account-profile/spec.md): owns `AccountOverview`, `AccountPage` and the account `page.tsx`. The Order History half of the overview (heading, empty state, `SECTION_IDS.orders`) moves into this spec's `OrderHistorySection`.
- **[Related]** [2026-10-14-wishlist](../2026-10-14-wishlist/spec.md): the same delegation pattern (`WishlistSection` replaced the wishlist half). This branch is stacked on it.

## Upstream Meeting

_None._

## Similar Implementations

### Owner-only order reads

- **Location:** `cms/src/api/order/controllers/order.ts` (`requireUser`, `findOne`), `order-store.ts` (`findByNumber`, `toOrderResponse`)
- **Relevance:** `listForUser` is `findByNumber` without the number, so it gets the same owner filter and view.

### Order snapshot → view

- **Location:** `frontend/src/components/checkout/checkout.ts` (`toOrderView`, `lineMeta`, `shippingLabel`), `OrderConfirmation.tsx`
- **Relevance:** the line, image and totals formatting from the snapshot; `formatPrice` and `strapiMediaUrl`.

### Account section delegation

- **Location:** `frontend/src/components/wishlist/WishlistSection.tsx`, `components/account/AccountOverview.tsx`
- **Relevance:** a section component rendered by `AccountOverview`, fed by a server fetch in the account page.

## Visuals

- `visuals/order-history-notes.md`: design → spec mapping from `westline-site-desing/profile.html` `#panelOrders`, with the gaps.

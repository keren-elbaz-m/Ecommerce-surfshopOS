# References for Checkout

## Related Specs

- **[Related]** [storefront-catalog/2026-10-10-add-to-cart](../../../storefront-catalog/specs/2026-10-10-add-to-cart/spec.md) owns the Strapi cart, the cart API, `CartProvider`, `lib/shipping.ts` and the drawer and `/cart`. Checkout builds on that work and touches its area in these ways:
  - It wires the Checkout CTAs, which supersedes its AC15.
  - `CartView` gains raw `subtotalAmount` / `unitAmount`.
  - `cart-store` gains `clearLines` and `purgeAbandoned`.
  - `identify()` is extracted to `cms/src/api/cart/identity.ts`.
  - A guest cart-creation rate limit is added to `app/api/cart/lines` POST.
  - It owns and fixes add-to-cart's carried tech debt (abandoned guest carts, unthrottled cart creation).

  The two stay separate because the cart is a storefront concern and ordering is cart-checkout's.
- **[Related]** [customer-account-wishlist/2026-09-22-account-signup-login](../../../customer-account-wishlist/specs/2026-09-22-account-signup-login/spec.md):
  - Login, register and Google OAuth honour a safe `?next=` (new `lib/auth/next-path.ts`, plus a small Google start route `app/api/auth/google/route.ts`).
  - `cms/config/server.ts` gains `cron`.
  - `getSession` / `requireAuth` are reused.

  It stays separate because only minimal edits are made to the auth exception area.
- **[Related]** seller-dashboard candidates `order-status-workflow`, `order-fulfillment-view` and `inventory-order-consistency-check` ([tech-plan](../../../seller-dashboard/tech-plan.md)) will extend this spec's Order model and status enum.
- **[Related]** customer-account-wishlist candidate `order-history` ([tech-plan](../../../customer-account-wishlist/tech-plan.md)) reads these Orders. Every order has a User, because checkout requires sign-in.

## Visuals

- `westline-site-desing/checkout.html`: the form plus a hidden `.is-confirmed` confirmation state. See [visuals/checkout-notes.md](visuals/checkout-notes.md).
- `westline-site-desing/profile.html` (order history) and `seller-orders.html` (order statuses and number format) were used for the model.

## Similar Implementations

### Strapi cart API

- **Location:** `cms/src/api/cart/` (`routes/cart.ts`, `controllers/cart.ts`, `cart-store.ts`, `cart-logic.ts`), `cms/tests/cart/*`
- **Relevance:** Order follows the same shape.
- **Key patterns:**
  - Custom routes with `auth: false` and no core router.
  - Identity is resolved in the controller (Bearer JWT via the users-permissions `jwt` service).
  - Pure, unit-tested logic.
  - An injectable store (`createCartStore(strapi, newToken)`) tested with a hand-rolled fake strapi.
  - `CartError` → status mapping.

### Emails

- **Location:** `cms/src/bootstrap/welcome-email.ts`, `cms/src/bootstrap/email-layout.ts` (`buildEmailHtml`), `cms/config/plugins.ts` (nodemailer → Resend SMTP)
- **Relevance:** The order confirmation email reuses the layout and the send path, wrapped in try/catch so a failed send never breaks the main flow.

### Next BFF for per-shopper calls

- **Location:** `frontend/src/lib/cart/route-helpers.ts`, `frontend/src/app/api/cart/**`, `frontend/src/lib/strapi/client.ts` (`strapiRequest`), `frontend/src/lib/strapi/cart.ts`
- **Relevance:** `/api/checkout` follows the same pattern.
- **Key patterns:**
  - Body validation before forwarding.
  - `no-store`; Strapi down → 503.
  - A 401 with a dead session triggers the `x-westline-session: expired` handling.

### Cart UI and tests

- **Location:** `frontend/src/components/cart/*` (`cart.ts` mapper, `cart-texts.ts`, `CartProvider.tsx`, `CartSummary.tsx`), `frontend/src/__tests__/components/cart/cart-test-utils.tsx`
- **Relevance:** The checkout summary and the wiring follow these.
- **Key patterns:**
  - The mapper sits next to its component.
  - Texts files.
  - `renderWithCart` for provider-backed component tests.

### Hook precedent

- **Location:** `frontend/src/components/product-detail/shared/use-add-to-cart.ts`
- **Relevance:** This is the precedent for keeping submit state in a hook (`use-place-order.ts`).

### Auth forms

- **Location:** `frontend/src/app/[locale]/auth/**` (`auth-field.tsx`, `utils.ts handleSubmit`)
- **Relevance:** The input styling is borrowed for `FormField`. The auth area itself is not restructured, because it is an exception under the frontend/components standard.

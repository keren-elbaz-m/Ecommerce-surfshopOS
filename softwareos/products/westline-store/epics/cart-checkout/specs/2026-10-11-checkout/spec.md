# Checkout

> Epic: ../../epic.md
> Status: done
> Branch: feat/cart-checkout/checkout
> Covers: /checkout, /checkout/confirmation/[orderNumber], frontend/src/app/[locale]/checkout/**, frontend/src/app/api/checkout/**, frontend/src/components/checkout/**, frontend/src/shared/components/form-field/**, frontend/src/lib/strapi/orders.ts, frontend/src/lib/strapi/account.ts, frontend/src/lib/checkout/**, frontend/src/lib/rate-limit.ts, cms/src/api/order/**, cms/src/components/order/**, cms/src/bootstrap/order-admin.ts, cms/config/cron-tasks.ts, cms/tests/order/**, frontend/src/__tests__/components/checkout/**, frontend/src/__tests__/lib/checkout/**, frontend/src/__tests__/app/api/checkout/**, frontend/src/__tests__/app/[locale]/checkout/**, frontend/src/__tests__/lib/strapi/orders.test.ts, frontend/src/__tests__/lib/strapi/account.test.ts, frontend/src/__tests__/lib/rate-limit.test.ts, frontend/src/__tests__/shared/components/form-field/**

## Overview

This spec completes Cart → Checkout → Confirmation. It merges three of the epic's tech-plan candidates: `order-content-model-admin-view`, `guest-checkout-flow` (now signed-in checkout) and `order-confirmation`. It builds on add-to-cart's Strapi cart, cart API and `CartProvider`. Real payment and real shipping integrations are out of v1.

**Design:** `westline-site-desing/checkout.html`. The confirmation is the page's `.is-confirmed` state, served at its own URL. See [visuals/checkout-notes.md](visuals/checkout-notes.md).

## Goals & User Stories

- As a shopper with items in my cart, I click Checkout, sign in if I'm not signed in yet, and come back to checkout with my cart intact.
- As a shopper, I enter my US shipping address, choose Standard or Express, and see my order total before paying.
- As a shopper, I pay with a (simulated) card and get a clear answer: the order is placed, or my card was declined and I can try again.
- As a shopper, I'm never charged a price I wasn't shown, and I can't order something that sold out while I was checking out.
- As a shopper, I see an order confirmation with my order number and details, and I get it by email.
- As the store, every order is recorded in Strapi with what was bought at what price, stock is decremented exactly once, and no double click creates two orders.

## Acceptance Criteria

1. **AC1: Order model** (schema JSON is approved before it is created).
   - New collection `api::order.order`, `draftAndPublish: false`, visible in the admin Content Manager.
   - Fields:
     - `OrderNumber`: string, unique, required, `WL-` + 6 digits.
     - `Status`: enum `processing | shipped | delivered | cancelled`, default `processing`.
     - `User`: manyToOne → user, required.
     - `Email`.
     - `ShippingAddress`: component `order.address` with FirstName, LastName, Address1, Address2?, City, State (2-letter), PostalCode, Country (`US`), Phone?.
     - `ShippingMethod`: enum `standard | express`.
     - `Lines`: repeatable component `order.line`, a snapshot with ProductDocumentId, Product (manyToOne, optional, reference only), Name, Slug, SizeKey, SizeLabel, UnitPrice, Quantity, LineTotal, ImageUrl, ImageAlt.
     - `Subtotal`, `ShippingCost`, `Total`: decimal.
     - `Currency`: `USD`.
     - `PaymentBrand`: enum `visa | mastercard | amex`.
     - `PaymentLast4`.
     - `PaymentReference`: `mock_<uuid>`.
     - `IdempotencyKey`: string, unique, private.
     - `PlacedAt`.
   - No core CRUD routes or public permissions.
   - **Read-only in the admin:** every field except Status, ShippingAddress and User is greyed out in the Order edit view (and every field inside a line); IdempotencyKey is hidden. "Create new entry" is removed (the content-manager create permission on Order is deleted on boot). Server-side, a document-service guard rejects any create or clone that doesn't come from the order store, and any update that changes a locked field (400, "These order fields are read-only: …").
   - The Product, Cart and User schemas are unchanged.
2. **AC2: Order number.** `WL-` + `crypto.randomInt(100000, 1000000)`, retried on a unique collision (max 5). The DB id is never exposed in URLs.
3. **AC3: Place-order API** `POST /api/orders` (`auth: false`; identity is checked in the controller like the cart).
   - A Bearer JWT is required → 401 without one.
   - Body: `{ idempotencyKey (uuid), shipping {…}, shippingMethod, card { number, expMonth, expYear, cvc, name }, expected { total, lines[{ productDocumentId, sizeKey, quantity, unitPrice }] } }`.
   - Validation failures → 400 `{ error: { code, message, fields? } }`.
4. **AC4: Re-validation at placement.**
   - The user's cart is read live (same revalidate as the cart).
   - An empty cart → 409 `cart-empty`.
   - 409 `cart-changed` when any of these differ from `expected`: the lines, a qty, a unit price or the total; or any line is `soldOut`, `unavailable` or `adjusted`; or `removedCount > 0`.
   - Shipping is recomputed on the server (AC10). Nothing is written.
5. **AC5: Atomic stock + payment + order.** Everything runs in one `strapi.db.transaction`, in this order:
   1. For each line, a conditional decrement `UPDATE <size table> SET stock = stock - q WHERE id = ? AND stock >= q`. The table and column come from `strapi.db.metadata`. 0 rows → rollback → 409 `cart-changed` with the sold-out line keys.
   2. Mock charge (AC7). Declined → rollback → 402 `card-declined`; processing error → rollback → 502 `payment-error`.
   3. Create the Order.
   4. Empty the cart's Lines.
   5. Commit → 201 `{ orderNumber }`.

   On any failure, no stock changes and no order exists.
6. **AC6: Idempotency.**
   - An order that already has this user + `idempotencyKey` → 200 with the same `{ orderNumber }`. There is no second charge and no second decrement.
   - A concurrent duplicate that hits the unique constraint rolls back and returns the existing order.
   - The client keeps one key per checkout attempt in sessionStorage and rotates it only after success.
   - Place Order is disabled while the request is pending.
7. **AC7: Mock payment** (`order/mock-payment.ts`, pure).
   - Luhn check.
   - Brand by prefix: Visa 4, Mastercard 51–55 / 2221–2720, Amex 34/37; any other → 400 `card-unsupported`.
   - CVC is 3 digits (4 for Amex). The expiry month/year must not be in the past. The name is required.
   - `4000000000000002` → declined. `4000000000000119` → error. Any other valid number → approved.
   - Only the brand, last 4 and reference are stored. The card number and CVC are never persisted or logged, in Strapi or in Next.
8. **AC8: Order read API** `GET /api/orders/:orderNumber`.
   - Bearer required.
   - Returns the order only to its owner. Not found or not owned → 404, the same response for both.
9. **AC9: Checkout page `/checkout`.**
   - **Server page:** dynamic, noindex, title "Checkout — WESTLINE".
     - No session → redirect `/auth/login?next=/checkout`.
     - An empty cart (client) → redirect to `/cart`.
   - **Layout:** a 58/42 split. The form sits in a white column with a 640px inner width. The summary is on `bg-surface` with a left border, sticky at `top: 96px`. Below 900px the columns stack with the summary underneath; below 520px each row stacks.
   - **Contact:** "Signed in as {email}" (read-only).
   - **Delivery:**
     - Country is shown as "United States" (fixed).
     - First / Last name, prefilled from the account.
     - Address, then "Apartment, suite, etc. (optional)".
     - City; State `<select>` (50 states + DC); ZIP (`12345` or `12345-6789`).
     - Phone (optional; 10 digits if given).
   - **Shipping Method:** radio cards, "Standard Shipping · Arrives in 5–7 business days · $12 / Free" and "Express Shipping · Arrives in 2–3 business days · $25".
   - **Payment:**
     - The note "All transactions are simulated…", plus the hint "Demo: use 4242 4242 4242 4242, any future date, any CVC."
     - The credit-card header with brand logos.
     - Card number (grouped as you type), Expiration (MM/YY), Security code, Name on card.
     - "Place Order": a dedicated submit button (design `.btn-place-order`). While pending it reads "Processing Payment…" with a spinner.
     - Then the fine print.
   - **Order Summary:** live cart lines (76×92 image, name, size label, "· Qty N", line total in mono), Subtotal, Shipping (by method), Total, and the note "Taxes are not included in this demo."
   - **Hidden:** discount code, the billing checkbox and the qty-index badge.
10. **AC10: Shipping rules.**
    - Standard costs $12, or $0 when the subtotal is at least `FREE_SHIPPING_THRESHOLD` (75). Express costs $25.
    - All money is computed in integer cents.
    - The rules live in `cms/src/api/order/shipping.ts` (authoritative) and `frontend/src/lib/shipping.ts` (display). Both are covered by the same test cases.
11. **AC11: Validation & a11y.**
    - Every input has a `<label>`.
    - Errors are shown inline under the field (`text-danger`), with `aria-invalid` and `aria-describedby`.
    - On submit, all fields are validated and focus moves to the first invalid field.
    - The Next route and cms re-validate everything.
    - Server field errors map back to their fields. Payment and cart errors show in a `role="alert"` area at the Payment section and take focus.
    - The whole flow works by keyboard only. Radio cards use native radios.
12. **AC12: Error states.**
    - **402:** "Your card was declined. Try another card." The card fields stay; everything else is kept.
    - **502:** "We couldn't process your payment. Try again."
    - **409 `cart-changed`:** reload the cart and show the banner "Your cart changed — review it and place your order again." Affected lines are marked: sold out, unavailable, "Only N left", or the new price.
    - **Blocking lines:** Place Order is disabled, with the note "Remove unavailable items to continue." and a link to the cart.
    - **429:** "Too many attempts. Wait a minute and try again."
    - **Network / 503:** a generic retry message.
    - In every case the shopper can retry without losing their shipping fields.
13. **AC13: Persistence across refresh.**
    - Shipping fields and the shipping method are saved to sessionStorage (`westline_checkout`; reads and writes wrapped in try/catch) and restored on load.
    - Card fields are never saved.
    - It is cleared after a successful order.
14. **AC14: Success.**
    - On 201 or 200, the client clears the sessionStorage key and rotates the idempotency key.
    - It calls `router.push` to the confirmation URL, then `CartProvider.reload()` (the badge goes to 0).
    - Browser Back from the confirmation lands on `/checkout`, which redirects to `/cart` (now empty).
15. **AC15: Confirmation page `/checkout/confirmation/[orderNumber]`.**
    - **Server page:** dynamic, noindex.
      - No session → redirect to login with `next` set to this URL.
      - An order that isn't owned or doesn't exist → `notFound()`.
    - **Content:** a check circle, "Thank You!", "Your order has been placed successfully.", "A confirmation has been sent to **{email}**.", and the "Order #WL-XXXXXX" pill.
    - **Order Summary card:** snapshot lines, Subtotal, Shipping ({method}), Total Paid; "Ship to" with the address; "Shipping method" with the ETA; "Payment" as "Visa •••• 4242".
    - **Action:** Continue Shopping → `PRODUCTS_HREF`.
    - All values come from the stored snapshot, never from live products.
16. **AC16: Confirmation email.**
    - Sent after commit through the existing email plugin and `buildEmailHtml`, fire-and-forget: the response never waits for SMTP. Subject: "Your WESTLINE order WL-XXXXXX".
    - It contains the lines, totals, address, method and a link to the confirmation URL (`CLIENT_URL`).
    - A send failure is logged and never affects the order response.
17. **AC17: Return to checkout after sign-in.**
    - Login and register accept `?next=`. It is honoured only for a same-site relative path (it must start with `/`, must not start with `//` or `/\`, and must have no scheme). Otherwise the shopper goes to `/`.
    - Google OAuth carries `next` through a short-lived httpOnly cookie, set before redirecting to Strapi and read in `oauth/complete`.
    - "Create an account" on login keeps `next`.
18. **AC18: Cart wiring.**
    - In the drawer and on `/cart`, Checkout becomes `ButtonCTA href=CHECKOUT_HREF`.
    - When `hasBlockingLines`, it renders disabled with the existing blocking note.
    - "Checkout is coming soon." is removed.
    - `CartView` gains raw `subtotalAmount` and a per-line `unitAmount`; the existing strings are unchanged.
19. **AC19: Abandoned guest carts.**
    - A Strapi cron runs daily at 03:00. It enables `cron` in `config/server.ts` with tasks from `config/cron-tasks.ts`.
    - It deletes carts with no User whose `updatedAt` is more than 30 days old (the cookie's lifetime), in batches through the document service.
    - The cutoff logic is pure and tested.
20. **AC20: Rate limiting.**
    - `lib/rate-limit.ts` in Next is an in-memory fixed-window limiter per client IP (`x-forwarded-for` first hop, then `x-real-ip`, then one shared bucket).
    - **Guest cart creation** (`POST /api/cart/lines` without a `westline_cart` cookie and without a session): 20 per 10 min.
    - **`POST /api/checkout`:** 10 per min per IP + user.
    - Exceeding either → 429 with `Retry-After`.
    - It is documented as single-instance only. Strapi called directly isn't covered (out of scope).
21. **AC21: Next BFF.**
    - `POST /api/checkout` and `GET` of the order are handled server-side. The confirmation page calls `lib/strapi/orders.ts` directly.
    - **POST /api/checkout:**
      - Reads `westline_session` → 401 when it's missing.
      - Validates with the shared `lib/checkout/validate.ts`.
      - Forwards the call through `strapiRequest`.
      - Returns `{ orderNumber, href }`, or the error code with `fields` / `cart: CartView`.
      - Uses `no-store`, and Strapi down → 503.
    - **Prefill** comes from `GET /api/users/me` through `lib/strapi/account.ts`.

## Technical Approach

**cms**
- **Schemas** (shown for approval before creation):
  - `src/api/order/content-types/order/schema.json`
  - `src/components/order/{line,address}.json`
  - Tables are auto-created. No migration and no Product change.
- **`src/api/order/routes/order.ts`:** custom routes only (POST /orders, GET /orders/:orderNumber).
- **`controllers/order.ts`:** identity uses the same JWT verify as cart. A good refactor is to extract `identify()` into `src/api/cart/identity.ts`, shared by both (Related: add-to-cart). It maps `OrderError` → status.
- **`order-logic.ts`** (pure): `validatePlaceOrder`, `compareWithExpected(cartResponse, expected)`, `toCents`, `buildOrderSnapshot(cart, shipping, method, payment)`, `orderNumber(rand)`.
- **`shipping.ts`:** `shippingCost(subtotalCents, method)`.
- **`mock-payment.ts`:** `luhn`, `cardBrand`, `charge(card, now)`.
- **`format.ts`:** `sizeLabel` (mirrors frontend `formatBoardSize` / `formatStandardSize`, with identical test cases) and `formatPrice` for the email.
- **`order-store.ts`** (`createOrderStore(strapi, deps)`, the same injectable style as `cart-store`):
  - `place(user, body)`: the transaction in AC5. The size row id is found by loading the product's size components inside the trx; the decrement uses `trx(table).where('id', id).andWhere(col, '>=', q).decrement(col, q)`.
  - `findByNumber(user, n)` and `findByIdempotencyKey(user, k)`.
- **`order-email.ts`:** the template plus `sendOrderEmail(strapi, order)`, wrapped in try/catch. The controller starts it after commit and doesn't await it.
- **`order-guard.ts`:** document-service middleware (registered in `src/index.ts` `register`) enforcing the read-only fields; the order store marks its own writes with `asOrderStore` (AsyncLocalStorage).
- **`src/bootstrap/order-admin.ts`:** on every boot, locks the Order and `order.line` edit views in the content-manager configuration (as `catalog.ts` does for Product) and removes the Order create permission.
- **Cart:** `src/api/cart/cart-store.ts` gains `purgeAbandoned(now)` and `clearLines(cartId, trx)`. Both are add-to-cart files and are recorded as Related.
- **`config/cron-tasks.ts`** + **`config/server.ts`** `cron: { enabled: true, tasks }`. `server.ts` is in account-signup-login's Covers, so it is Related.
- **Tests** (Vitest, TDD) in `cms/tests/order/*`: logic, shipping, mock-payment, format, the store with a fake strapi + fake trx, and the cart purge.

**frontend**
- **`lib/routes.ts`:** `CHECKOUT_HREF`, `CHECKOUT_API_HREF`, `orderConfirmationHref(n)`, `loginHref(next?)`.
- **`lib/shipping.ts`:** `STANDARD_SHIPPING`, `EXPRESS_SHIPPING`, `shippingCost(subtotal, method)`. The threshold is unchanged.
- **`lib/checkout/`:**
  - `validate.ts` (shipping and card validation shared by client and route; `US_STATES`; `luhn`; `cardBrand`; expiry)
  - `storage.ts` (sessionStorage, try/catch)
  - `place-order-body.ts`
- **`lib/strapi/orders.ts`:** `placeOrder(jwt, body)`, `getOrder(jwt, n)`. **`lib/strapi/account.ts`:** `getMe(jwt)`.
- **`lib/rate-limit.ts`**, used in `app/api/checkout/route.ts` and `app/api/cart/lines/route.ts` (POST).
- **`app/[locale]/checkout/page.tsx`** and **`checkout/confirmation/[orderNumber]/page.tsx`:** thin.
- **`components/checkout/`** (app-level):
  - `checkout-texts.ts`
  - `checkout.ts` (view types, `toOrderView` mapper)
  - `CheckoutPage.tsx` (grid, states, redirect-if-empty)
  - `CheckoutForm.tsx` (all sections inline; state lifted)
  - `CheckoutSummary.tsx`
  - `use-place-order.ts` (idempotency key, submit, error → field/alert mapping, focus-first-error)
  - `OrderConfirmation.tsx`
  - `index.ts`
- **`shared/components/form-field/FormField.tsx`:**
  - Reusable: label, input or select, hint, error, `aria-invalid` / `aria-describedby`, and the design's input styling (9px radius, `border-border`, Horizon focus ring).
  - It forwards a ref so the first error can take focus.
- **Icon:** add `check`.
- **Auth `next`:**
  - A new `lib/auth/next-path.ts` `safeNextPath()`.
  - Minimal edits to the login and register pages and the `oauth/complete` route, plus a cookie set by the Google button.
  - The structure is not changed (auth is an exception area).
- **Cart:** `CartDrawer.tsx` and `CartSummary.tsx` wire Checkout; `cart.ts` gains raw amounts. Their tests are updated.
- **Tests** (Vitest + RTL, TDD), mirrored under `src/__tests__/`: validate, shipping, storage, rate-limit, routes, orders client, the checkout route, CheckoutForm (errors, focus, submit states), CheckoutSummary, OrderConfirmation, FormField, safeNextPath, and the updated cart tests.

## Out of Scope
- Real payment gateways, 3DS, refunds. Real carriers, tracking, address verification.
- Tax.
- Non-US shipping.
- Discount codes.
- A separate billing address.
- Saving addresses to the account.
- Guest (no-account) checkout.
- An account-creation offer after ordering.
- Order history (customer-account-wishlist/order-history).
- Status workflow, fulfilment views and seller notifications (seller-dashboard).
- Cancelling an order or restocking on cancel.
- A distributed rate limiter. Rate limiting Strapi when it is called directly.
- A verified sending domain (Resend sandbox: emails only reach the account owner's address).
- The design's minimal checkout header: the regular SiteNav is kept.

## Known Limitations
- **The mock charge runs inside the DB transaction** (AC5). A real gateway must move the charge outside it. The sequence then becomes:
  1. Reserve or decrement stock and create the order as `pending` (or validate only).
  2. Charge.
  3. Commit or mark the order paid.
  4. If anything after the charge fails, issue a compensating refund or void.

  Holding a DB transaction open across a network call to a payment provider isn't acceptable in production.
- **The stock decrement assumes Product has `draftAndPublish: false`** (verified 2026-10-11). Each size is one component row. If draft & publish is ever enabled, the published row must be decremented conditionally and the matching draft row decremented by the same amount in the same transaction. Otherwise admin shows stale stock and a later Publish restores the old value.

## Standards Applied
- [frontend/components](../../../../../../standards/frontend/components.md): one component and one return per file, texts files, `app/` routing only, Strapi only via `client.ts`, URLs via `routes.ts`, mapper next to component, `Icon`.
- [global/git-workflow](../../../../../../standards/global/git-workflow.md): branch `feat/cart-checkout/checkout`, scoped commits.

## Changelog
| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-10-11 | Ori Chai Matan | created | Initial shaping | — |
| 2026-10-11 | Ori Chai Matan | change | Built T2–T13. AC6: concurrent duplicates are serialised by locking the user's cart row (`SELECT … FOR UPDATE`) at the start of the place-order transaction and re-checking the idempotency key under the lock, not by catching a unique violation (a failed insert aborts a Postgres transaction, and Strapi enforces `unique` in the application). AC2: order-number collisions are pre-checked inside the transaction. AC7: an unsupported brand is a field error (400 `bad-request`, `fields.cardNumber`) rather than its own `card-unsupported` code. `identify()` returns the user's email too. Strapi runs on Postgres locally | tasks.md T5–T13 |
| 2026-10-11 | Ori Chai Matan | change | Checkpoint 1 (T14) passed live; data restored (see tasks.md T14) | tasks.md T14 |
| 2026-10-11 | Ori Chai Matan | change | Before T15: Product `draftAndPublish` re-checked, still `false`, so the decrement updates the one size row (Known Limitations unchanged) | — |
| 2026-10-11 | Ori Chai Matan | change | AC9: Place Order is a dedicated submit button (design `.btn-place-order`: full width, spinner, "Processing Payment…"), not ButtonCTA. The ButtonCTA `type` prop was built test-first, then reverted as unused (T25 cancelled; the button-cta Changelog row was removed). The `lock` icon was dropped (the design has none); only `check` was added. The design's "+5" card chip is omitted (only Visa / Mastercard / Amex are accepted). Country is fixed text; Phone is labelled "(optional)"; the demo hint also names the declined test card | tasks.md T25, T26, T32 |
| 2026-10-11 | Ori Chai Matan | change | AC14: success navigates with `router.push`, not `replace`, so browser Back lands on `/checkout`, which sends the now-empty cart to `/cart` (with `replace`, Back skipped to the page before checkout). Test first | tasks.md T34, T39 |
| 2026-10-11 | Ori Chai Matan | change | AC20: the limiter's IP falls back from `x-forwarded-for` to `x-real-ip`, then one shared bucket (route handlers get a plain `Request`). AC12: cart-empty shows "Your cart is empty." and reloads (the page then redirects to `/cart`); network and 503 show "We couldn't place your order. Check your connection and try again."; a 401 sends the shopper to sign in with `next=/checkout` | tasks.md T20, T34 |
| 2026-10-11 | Ori Chai Matan | change | AC17: login and register read `next` from the page's `searchParams` prop (no Suspense needed); Google goes through a new `app/api/auth/google` route that sets `westline_next` (httpOnly, 10 min), and `oauth/complete` returns `next` to the redirect page. AC18: `CartView.subtotalAmount` / `unitAmount` were added in T17, ahead of T38, because the place-order body is built from them | tasks.md T17, T24 |
| 2026-10-11 | Ori Chai Matan | change | Design fidelity: inputs use `leading-[1.2]` to match the design's 43px height (Tailwind preflight made them 48px). Summary column uses `bg-surface` (#F9F9F9) rather than the design's #f5f5f5; the regular SiteNav is kept | qa-report.md |
| 2026-10-11 | Ori Chai Matan | change | Built T15–T38; Checkpoint 2 (T39) self-run in the browser, 33/33; QA (T42) PASS WITH NOTES (qa-report.md); all test data restored (T43). Status → done | qa-report.md |
| 2026-10-11 | Ori Chai Matan | fix | Placing an order was slow: measured per step, the awaited confirmation email (SMTP to Resend) took 928–1671ms warm and 7260ms on the first order after a restart, while everything else (JWT, validation, cart lookup, the transaction with lock + cart read + decrement + mock charge + order create + cart clear) took 39–68ms. The mock charge has no delay. The controller now sends the email fire-and-forget after the response (failures still logged), test first. Result: Strapi answers in 23–52ms (52ms first after restart), Next `/api/checkout` in 30–52ms, and the shopper reaches the confirmation page 105–114ms after clicking Place Order (212ms on the first visit). No client-side minimum for "Processing Payment…" was added: it shows for the ~60ms the request takes | qa-report.md |
| 2026-10-11 | Ori Chai Matan | change | AC1: Order fields are read-only in the admin except Status, ShippingAddress and User. `src/bootstrap/order-admin.ts` sets them non-editable in the Order edit view, hides IdempotencyKey, makes every field of `order.line` non-editable (Lines alone only greyed the add/delete controls), and deletes the content-manager create permission on Order so "Create new entry" is gone, even for Super Admin (Strapi re-grants it on boot, so it is removed on every boot). `order-guard.ts` enforces it server-side as document-service middleware: create/clone outside the order store → 400 "Orders are created at checkout only."; an update changing a locked field → 400 "These order fields are read-only: …" (values compared by value, so a full admin save that only changes Status passes). Uses `errors.ValidationError` from `@strapi/utils` (a Strapi dependency, not listed in cms `package.json`). 19 new tests (guard, admin views, store flag) | tasks.md |
| 2026-10-11 | Ori Chai Matan | change | Verified live with a temporary Super Admin (created and deleted): Create new entry absent; OrderNumber, Email, ShippingMethod, Subtotal, ShippingCost, Total, Currency, PaymentBrand, PaymentLast4, PaymentReference, PlacedAt and all line fields greyed out; Add an entry / delete line disabled; IdempotencyKey hidden; Status changed and saved in the UI; ShippingAddress change saved via the admin API; Total / Email changes via the admin API → 400; create / clone via the admin API → 403. Checkout still places orders with the guard on. All test data restored (stock, 24 test orders, 7 test users and carts, the temporary admin); the shopper's own order WL-714224 was not touched | qa-report.md |
| 2026-10-13 | Ori Chai Matan | change | account-profile adds `updateMe` to `lib/strapi/account.ts` (one added function; `getMe` unchanged), tested in a second `describe` in `__tests__/lib/strapi/account.test.ts`. The new cms `api/account` controller reuses `createIdentify` from `api/cart/identity.ts` unchanged | customer-account-wishlist/2026-10-13-account-profile |
| 2026-10-14 | Ori Chai Matan | change | order-history: the order API gains an owner-only `GET /orders` (`order.list` → `listForUser`: the caller's orders, `PlacedAt` desc, mapped with the existing `toOrderResponse`); `lib/strapi/orders.ts` gains `getOrders(jwt)` (reuses `isOrder`, drops bad entries, null on failure). Tests added in `cms/tests/order/order-store.test.ts`, `order-controller.test.ts` (routes table now has three routes) and `__tests__/lib/strapi/orders.test.ts`. Nothing existing changed | customer-account-wishlist/2026-10-14-order-history |
| 2026-10-14 | Ori Chai Matan | change | Design-system pass (Ori's request, from account-profile): the shared FormField now renders the system's input — mono uppercase 12px label, 15px input — so checkout's fields change too; the hand-built Country/Region label in `CheckoutForm.tsx` matches it. No behaviour change | design system v0.4 (westline-site-desing/design-system.md) |

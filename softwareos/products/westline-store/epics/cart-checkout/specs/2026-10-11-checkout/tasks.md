# Tasks — Checkout
> Epic: cart-checkout
> Spec: 2026-10-11-checkout
> Branch: feat/cart-checkout/checkout


## Setup
- [x] T1 Branch feat/cart-checkout/checkout from feat/storefront-catalog/add-to-cart (clean tree). Record baseline cms + frontend test, lint and tsc counts — baseline: cms 43 tests (2 files), tsc clean; frontend 487 tests (54 files), lint warnings only, tsc clean

## Strapi order: pure logic
- [x] T2 Failing tests in cms/tests/order/:
  - shipping.test.ts: 0/7499/7500/20000 → 1200/1200/0/0, express 2500. Same cases as T15 (AC10)
  - mock-payment.test.ts: luhn; brands and their edges; CVC by brand; expiry; name; the decline/error/approve test numbers; the result never holds the number or CVC (AC7)
  - format.test.ts: sizeLabel matches the frontend size formatter cases; formatPrice (AC16)
  - order-logic.test.ts: orderNumber, validatePlaceOrder field errors, compareWithExpected (every cart-changed case and cart-empty), toCents, buildOrderSnapshot (AC2–AC4)
- [x] T3 Implement order/shipping.ts, mock-payment.ts, format.ts, order-logic.ts (pure)

## Strapi order: schema
- [x] T4 Draft order schema.json + components order/address.json and order/line.json. **STOP: show the schema JSON for approval before creating.** Then create them, boot Strapi, and confirm Order in the Content Manager. Confirm the Product, Cart and User schema diff is empty (AC1) — approved as drafted 2026-10-11; tables orders / components_order_addresses / components_order_lines created on Postgres; Product, Cart, User schemas unchanged

## Strapi order: store and transaction
- [x] T5 Extract identify() → cms/src/api/cart/identity.ts (no behaviour change). Add identity.test.ts (AC3)
- [x] T6 Failing order-store.test.ts (fake strapi, trx and metadata) (AC4–AC6, AC8):
  - success path; stale and empty carts; 0-row decrement → rollback + sold-out keys
  - decline/error rollback; idempotent replay; unique-key race; order-number retry
  - findByNumber owner scoping; clearLines in cart-store
- [x] T7 Implement order-store.ts (conditional knex decrement in strapi.db.transaction) + cart-store clearLines

## Strapi order: controller and routes
- [x] T8 Failing order-controller.test.ts: 401, 400 fields, OrderError→status, 201 vs 200 replay, identical 404 for another user's order and an unknown number, no id/key/card in responses, custom routes only (AC3, AC8)
- [x] T9 Implement routes/order.ts, controllers/order.ts, services/order.ts. No permissions granted and no core CRUD

## Strapi order: email
- [x] T10 Failing order-email.test.ts: subject, contents, link, no card data; a send that throws is logged; sent only after commit, never on errors or replays (AC16)
- [x] T11 Implement order-email.ts and call it from the controller after commit

## Strapi: abandoned guest carts
- [x] T12 Failing cart-purge.test.ts: 30-day cutoff boundaries, guest-only, batched, cron rule 0 3 * * * (AC19)
- [x] T13 Implement purgeAbandoned + config/cron-tasks.ts + cron in config/server.ts. Confirm it boots

## Checkpoint 1
- [x] T14 **CHECKPOINT 1 — STOP for review.** Record stock, orders and carts. Run the live curl matrix:
  - success; decline; error; sold-out race (Stock=1, two orders)
  - cart-changed; cart-empty; idempotent replay; 401
  - user B → A's order 404; no core CRUD
  Then restore everything and list each change (AC1–AC8, AC16)
  - Result 2026-10-11 (Postgres, local Strapi): decline → 402, error → 502, wrong total → 409 cart-changed, bad body → 400 with fields, no JWT → 401, success → 201 (WL-250098), same key → 200 same order (twice), new key on empty cart → 409 cart-empty, A reads own → 200, B reads A's → 404 identical to unknown, `/api/orders` CRUD → 404. Stock moved once (driftwood 7-6-48.5 4→3, tidal M 8→6); declined/errored attempts left it untouched. Race for coldwater S (stock 1): B 201, A 409 cart-changed, stock 0 (never negative). Concurrent identical requests: 201 + 200, same WL-199446, one decrement. Email attempted after commit; Resend sandbox rejected example.com (logged, order unaffected)
  - Data changed and restored: registered checkout-test-a/b@example.com (+ their carts), 3 orders (WL-250098, WL-604604, WL-199446), stock driftwood 7-6-48.5 4→3, tidal-rash-guard M 8→5, coldwater-4-3-chest-zip S 1→0. Restored: stock 4 / 8 / 1, orders deleted, both users and carts deleted. Before/after snapshot identical (0 orders, 2 carts, 2 users)

## Frontend: routes, shipping, checkout lib
- [x] T15 Failing routes.test.ts additions (CHECKOUT_HREF, CHECKOUT_API_HREF, orderConfirmationHref, loginHref(next)) + new shipping.test.ts (same cases as T2) (AC10, AC17, AC21)
- [x] T16 Implement the lib/routes.ts + lib/shipping.ts additions
- [x] T17 Failing lib/checkout tests: validate.test.ts, storage.test.ts (no card fields; idempotency key rotate; storage throwing), place-order-body.test.ts (AC3, AC4, AC6, AC7, AC11, AC13)
- [x] T18 Implement lib/checkout/validate.ts, storage.ts, place-order-body.ts — also pulled forward from T37/T38: `CartView.subtotalAmount` and `CartLineView.unitAmount` (test first in cart.test.ts), since the body is built from them

## Frontend: rate limit and Strapi clients
- [x] T19 Failing rate-limit.test.ts: window, per-key, XFF first hop, retryAfter (AC20)
- [x] T20 Implement lib/rate-limit.ts (single-instance note)
- [x] T21 Failing lib/strapi/orders.test.ts + account.test.ts (AC8, AC21)
- [x] T22 Implement lib/strapi/orders.ts + account.ts

## Frontend: auth next
- [x] T23 Failing tests: next-path.test.ts (open-redirect cases); login/register keep and honour next; oauth/complete reads and clears the next cookie; Google start route sets the cookie (AC17)
- [x] T24 Implement lib/auth/next-path.ts + minimal edits to the login and register pages, the Google button (via a new app/api/auth/google route) and oauth/complete

## Frontend: shared components
- [-] T25 ButtonCTA type prop (test first) — cancelled: Place Order is a dedicated submit button (design `.btn-place-order`: full width, spinner, "Processing Payment…"), which ButtonCTA has no slot for; the `type` prop was added test-first, then reverted as unused, and the button-cta Changelog row removed
- [x] T26 Icon check (+ lock only if used) (test first) — `check` added; `lock` dropped (the design has no lock icon)
- [x] T27 Failing FormField.test.tsx: label, hint, error, aria-invalid, aria-describedby, select, ref focus (AC11)
- [x] T28 Implement shared/components/form-field/FormField.tsx + index.ts

## Frontend: checkout API route
- [x] T29 Failing app/api/checkout/route.test.ts:
  - 401; 400 with no Strapi call; pass-through codes; cart-changed returns a CartView
  - 503; no-store; 429 + Retry-After; card data never logged
  - plus the guest cart-creation limit in cart/lines (AC3, AC12, AC20, AC21)
- [x] T30 Implement app/api/checkout/route.ts + the limiter in api/cart/lines POST

## Frontend: checkout page
- [x] T31 Failing tests: checkout.test.ts (toOrderView), CheckoutSummary.test.tsx, CheckoutForm.test.tsx (prefill, fixed US, 51 states, radio cards, hint, hidden elements, inline errors + focus-first-error, storage restore, blocking lines, keyboard), CheckoutPage.test.tsx (empty → /cart, layout) (AC9–AC13)
- [x] T32 Implement components/checkout/{checkout-texts.ts, checkout.ts, CheckoutSummary, CheckoutForm, CheckoutPage, index.ts} + app/[locale]/checkout/page.tsx (redirect to login, getMe prefill)
- [x] T33 Failing use-place-order.test.tsx: key reuse/rotation, pending lock, success flow, field errors + focus, every AC12 message, cart-changed (AC6, AC11, AC12, AC14)
- [x] T34 Implement use-place-order.ts and wire it

## Frontend: confirmation page
- [x] T35 Failing OrderConfirmation.test.tsx + confirmation page test (login redirect with next, notFound, snapshot only) (AC15)
- [x] T36 Implement OrderConfirmation.tsx + app/[locale]/checkout/confirmation/[orderNumber]/page.tsx

## Frontend: cart wiring
- [x] T37 Update failing cart tests: raw amounts in cart.ts; Checkout link and blocked state in Drawer, Summary and Page; coming-soon copy gone (AC18)
- [x] T38 Implement cart.ts raw amounts, CartDrawer + CartSummary Checkout, cart-texts cleanup

## Checkpoint 2
- [x] T39 **CHECKPOINT 2 — STOP for review.** Record data, then run the browser E2E: — self-run 2026-10-11 (Playwright, real cookies): 33/33 passed; guest → login?next → register → merged cart → refresh keeps draft → decline → cart changed elsewhere → retry → confirmation → badge 0 → Back → /cart; B on A's URL → 404; sold-out race one 201 / one refused with "Sold out". Google sign-in not run (needs a real account). Data restored, see qa-report.md
  - guest add (board + standard) → Checkout → login?next → back with the merged cart (also via register and Google)
  - reload keeps the shipping fields; decline then retry; success → confirmation
  - email; badge 0; Back → /cart; stock decremented once; B → 404
  Then restore and record (AC9–AC18)

## Verification
- [x] T40 Full suites (testing skill): every AC has at least one test. lint, tsc (both), frontend build. Compare with the T1 baseline — cms 192 tests (+149), frontend 670 (+183), lint 0 errors (1 pre-existing warning), tsc clean both, `next build` passes (built from a scratchpad copy so `next dev` was untouched)
- [x] T41 Browser regression: cart drawer, /cart, surfboard PDP, standard PDP (add, stepper, remove, sold-out/unavailable, focus trap, Checkout link and disabled state) — passed (drawer focus trap / Esc / focus return, PDP adds, /cart stepper + Remove, sold-out state, Checkout link and disabled state)
- [x] T42 QA (qa skill) at 1440/900/480/390 against westline-site-desing/checkout.html (form + confirmation) → qa-report.md — PASS WITH NOTES, qa-report.md
- [x] T43 Data-restore report in qa-report.md (every value before and after) — qa-report.md "Data changed for testing and restored"
- [x] T44 Reconcile the Related Changelog rows + tech-plan pointers; add divergence rows; regenerate specs.index.yml — Changelog rows added; button-cta row removed (T25 cancelled); specs.index.yml regenerated
- [x] T45 Status → done in spec.md and epic.md

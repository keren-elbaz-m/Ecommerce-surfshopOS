# QA Report — Checkout

> Spec: spec.md
> Date: 2026-10-11 · QA by: Ori Chai Matan · Mode: setup
> Verdict: PASS WITH NOTES

## Acceptance Criteria

| AC | Criterion | Result | Evidence |
|---|---|---|---|
| AC1 | `api::order.order` + `order.address` / `order.line` with snapshots; no core CRUD or permissions; Product / Cart / User schemas unchanged | pass | Schema approved before creation (T4). Tables `orders`, `components_order_addresses`, `components_order_lines` exist on Postgres. `git diff cms/src/api/product cms/src/api/cart/content-types cms/src/extensions cms/src/components/{product,cart}` is empty. Live: `GET /api/orders` → 404. Snapshot written by `buildOrderSnapshot` (`order-logic.test.ts`), with no card number or CVC fields. The admin Content Manager listing is **not verified** (no admin credentials; see Notes). |
| AC2 | `WL-` + 6 digits; collisions retried; DB id never exposed | pass | `order-logic.test.ts` orderNumber; `order-store.test.ts` "retries a taken order number" and "fails after 5 taken order numbers". Live order numbers: WL-250098, WL-604604, WL-199446, WL-715240, WL-901647. `toOrderResponse` omits id, documentId, IdempotencyKey and PaymentReference (test). |
| AC3 | `POST /api/orders`: Bearer required, body validated, 400 with fields | pass | `order-controller.test.ts` (401 for none / guest / bad JWT, 400 with fields before the store runs). Live curl (T14): no JWT → 401; malformed body → 400 with every field error. |
| AC4 | Live re-validation; `cart-empty`; `cart-changed` on any difference; nothing written | pass | `order-logic.test.ts` compareWithExpected (11 cases); `order-store.test.ts` "rolls back with cart-changed … touching no stock". Live: wrong total → 409 `cart-changed`; empty cart → 409 `cart-empty`. Browser: a quantity changed in another tab → refused with "Your cart changed — review it and place your order again.", the line marked "Price or quantity updated." with the new quantity. |
| AC5 | One transaction: conditional decrement → mock charge → order → empty cart; any failure rolls back | pass | `order-store.test.ts` asserts the exact sequence (lock, `decrement … where stock >= q` per size row, create, clear, commit) and a rollback for a 0-row decrement, decline, processing error, unexpected error and order-number exhaustion. Live (Postgres): declined and errored attempts left stock untouched; every successful order moved stock exactly once (see Data). Race for coldwater S (Stock 1), via curl and again in the browser: one 201, the other 409 `cart-changed` with that line key; stock stopped at 0, never negative. |
| AC6 | Idempotency: same key → same order, no second charge / decrement; concurrent duplicate; key per attempt; button disabled while pending | pass | `order-store.test.ts` (replay without a transaction; replay found under the cart lock). Live: the same key twice → 200 with the same WL-250098; two **concurrent** identical requests → 201 + 200, one order (WL-199446), one decrement. `use-place-order.test.tsx`: one key reused across retries, rotated only after success; a double submit sends one request. |
| AC7 | Mock payment: Luhn, brands, CVC, expiry, name; test numbers; only brand / last 4 / reference stored, never logged | pass | `mock-payment.test.ts` (36) and `validate.test.ts`. Live: `4000 0000 0000 0002` → 402, `4000 0000 0000 0119` → 502, `4242…` → 201 with `payment: { brand: 'visa', last4: '4242' }`. `order-controller.test.ts` and `route.test.ts` check that responses and logs never contain the card number or CVC. |
| AC8 | `GET /api/orders/:orderNumber`: owner only; unknown and not-owned are the same 404 | pass | `order-controller.test.ts` (byte-identical 404s). Live curl and browser: user B on A's order → 404; unknown number → 404, same body. |
| AC9 | `/checkout`: guest → login?next; empty cart → /cart; 58/42 layout; sections and fields; radio cards; payment box; summary; hidden design items | pass | `CheckoutForm.test.tsx` (13), `CheckoutSummary.test.tsx` (5), `CheckoutPage.test.tsx` (6), page test (4). Browser vs `checkout.html`: split 0.580 ours vs 0.580 design at 1440; form inner 640px; summary sticky at `top: 96px` with a 1px left border; stacked with a top border at 900 / 480 / 390; rows stack below 520px; 16px gutter at ≤480; inputs 43.4px vs the design's 43px; no horizontal scroll at any width. Production build: guest `/checkout` → 307 to `/auth/login?next=%2Fcheckout`. |
| AC10 | Standard $12 / free from $75; Express $25; cents; same cases on both sides | pass | `cms/tests/order/shipping.test.ts` and `frontend/src/__tests__/lib/shipping.test.ts` run the same 7 cases. Browser: "Free" above $75, $25 for Express in the option and the total. |
| AC11 | Labels, inline errors with `aria-invalid` / `aria-describedby`, focus first error, server errors mapped, keyboard-only | pass | `FormField.test.tsx` (7); `use-place-order.test.tsx` focus and server-field mapping. Browser: submit with empty fields focused `address1` (first invalid field, after the prefilled names); ZIP marked `aria-invalid`. Keyboard-only pass: Tab order firstName → … → phone → shipping radio → card fields → Place Order; arrow keys switch the method; State picked by typing; Enter submits; the alert takes focus; inputs show the Horizon focus ring. |
| AC12 | Error states: 402, 502, cart-changed, blocked, 429, network | pass | `use-place-order.test.tsx` checks each message and that every field and the key survive. Browser: decline message + fields kept + alert focused; cart-changed banner and marks; a sold-out line disables Place Order with "Remove unavailable items to continue. Review your cart". 429 is covered by `route.test.ts` (11th attempt). |
| AC13 | Shipping draft survives refresh; card fields never stored; cleared after an order | pass | `storage.test.ts` (6, including storage that throws). Browser: reload kept City, State and Express; Card number came back empty. |
| AC14 | Success: draft cleared, key rotated, cart reloaded, confirmation URL; Back → /checkout → /cart | pass | `use-place-order.test.tsx`. Browser: confirmation URL, header badge back to "Cart" (0), Back → `/cart` showing "Your cart is empty." Navigation is `router.push`, not `replace` (Changelog). |
| AC15 | Confirmation page: owner only, snapshot content, Continue Shopping | pass | `OrderConfirmation.test.tsx` (4), `checkout.test.ts` toOrderView, page test (5). Browser at 1440 / 900 / 480 / 390: column ≤ 640px and centred, no horizontal scroll; shows Thank You!, the email, `Order #WL-715240`, both lines, "Shipping (Standard Shipping) Free", "Total Paid $897", ship-to, method + ETA, "Visa •••• 4242". |
| AC16 | Confirmation email after commit; failures logged, never fail the order | pass with note | `order-email.test.ts` (8): subject, lines, totals, address, method, link, HTML escaping, no card data, logged failure. `order-controller.test.ts`: sent only after a created order, never on errors or replays. Live: a send was attempted after each of the 5 orders; Resend's sandbox rejected the `@example.com` test recipients (550), which was logged, and every order still succeeded. **Actual delivery is unverified** (see Notes). |
| AC17 | Safe `?next=` on login, register and Google | pass with note | `next-path.test.ts` (16 incl. open-redirect cases), `login/next.test.tsx` (5), `register/next.test.tsx` (2), `google/route.test.ts` (2), `oauth/complete` (2). Browser: login → "Create an account" kept `next`; register returned to `/checkout`; sign-in with `?next=/cart` returned to `/cart`. **Google sign-in itself is unverified live** (needs your Google account). |
| AC18 | Cart wiring: Checkout links to /checkout, disabled with blocked lines, coming-soon gone; raw amounts | pass | `cart.test.ts`, `CartDrawer.test.tsx`, `CartSummary.test.tsx`. Browser: drawer and summary link to `/checkout`; disabled with a sold-out line; no "coming soon" note. |
| AC19 | Daily 03:00 purge of guest carts older than 30 days | pass | `cart-purge.test.ts` (5: cutoff, guest-only filter, batching, cron rule). Strapi boots with `cron.enabled` and the task registered. A real purge run was not triggered (it would delete real guest carts; see Notes). |
| AC20 | Rate limits: guest cart creation 20 / 10 min; place order 10 / min; 429 + Retry-After | pass | `rate-limit.test.ts` (6), `route.test.ts` (11th attempt → 429), `lines/route.test.ts` (21st new guest cart → 429; cookie or session holders never limited). |
| AC21 | Next BFF: 401, validation, forwarding, mapping, no-store, 503; prefill via users/me | pass | `app/api/checkout/route.test.ts` (14), `orders.test.ts` (12), `account.test.ts` (3). Live: prefill showed the account name and email. Production build: `POST /api/checkout` without a session → 401. |

## Checks Run

Compared with the T1 baseline:

| Check | Baseline (T1) | Now |
|---|---|---|
| cms tests (`npm test`) | 43 passed, 2 files | 192 passed, 11 files (+149) |
| cms typecheck (`npx tsc --noEmit -p .`) | clean | clean |
| frontend tests (`npm test`) | 487 passed, 54 files | 670 passed, 75 files (+183) |
| frontend lint (`npm run lint`) | 0 errors, 1 warning | 0 errors, the same 1 pre-existing warning (`no-page-custom-font`) |
| frontend typecheck (`npx tsc --noEmit`) | clean | clean |
| frontend build (`next build`) | not run | passes. It was built from a scratchpad copy so your `next dev` `.next` stayed intact. Smoke-tested on :3100, then stopped. |

- **Browser:** Playwright 1.63 (npx cache) with headless Chromium, against `strapi develop` (Postgres) and `next dev`. Results:
  - E2E (T39): 33/33 passed.
  - QA (T41/T42): 50/51 on the first run. The one miss was a timing issue in the script (`isVisible()` doesn't wait while `/cart` loads); it passed on a re-check with waits (below).
  - Screenshots are in the session scratchpad and not committed.
- **Live API (T14):** a curl matrix against Strapi, recorded in tasks.md.

## Issues Found

Found and fixed during verification:
- **Back from the confirmation skipped `/checkout`.** It used `router.replace`, so Back went to the page before checkout (login/register). It now uses `router.push`, so Back lands on `/checkout`, which sends the now-empty cart to `/cart`, as the spec intends. Test first.
- **Next's generated page types rejected `= {}` on the login and register `searchParams` props** (found by `tsc` against `.next/types`). The default was removed.
- **Inputs were 48px tall vs the design's 43px** (Tailwind preflight `line-height: 1.5`). Set to `leading-[1.2]`, now 43.4px.
- **Lint:** three unused variables (a test import, two destructuring leftovers). Fixed.

## Notes

- **Unverified, needs you:**
  - **Email delivery.** Resend's sandbox (`onboarding@resend.dev`) only delivers to the Resend account owner's address. Place one order signed in with that address to see the email.
  - **Google sign-in with `?next=`.** It needs a real Google account and `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET`. Unit-tested only.
  - **The Order collection in the Strapi admin Content Manager.** I have no admin credentials. It's a standard collection type with default visibility, so it should list under "Order".
  - **The 03:00 cron purge running on real data.** I didn't trigger it, because it would delete real abandoned guest carts. The logic is unit-tested.
- **Sold-out regression re-check:** I set tidal-rash-guard XL to 0 in user B's cart. `/cart` then showed "Sold out", Checkout disabled and the blocking note, and `/checkout` showed the line marked with Place Order disabled. Stock was restored (see Data).
- **Design differences** (recorded in the spec Changelog):
  - The regular SiteNav stays, not the design's minimal header.
  - The summary column uses `bg-surface` `#F9F9F9`, not the design's `#f5f5f5`.
  - The country is fixed text (US only).
  - Discount code, the billing checkbox, View Order, the qty-index badge and the "+5" card chip are hidden.
  - A demo-card hint line is added.
  - Place Order is a dedicated button, not ButtonCTA.
  - The order-number pill renders lighter than the design's 700 weight, probably because JetBrains Mono isn't loaded in bold.
- **Known limitations** (in the spec):
  - The mock charge runs inside the DB transaction.
  - The decrement assumes Product has `draftAndPublish: false`, which was verified again before T15.
  - The rate limiter is in-memory and covers a single instance only.

## Data changed for testing and restored (T43)

Every value was recorded before testing and restored afterwards. The before and after snapshots are identical both times.

**Checkpoint 1 (T14, curl)**

| Data | Before | During | After |
|---|---|---|---|
| driftwood-7-6-mid-length `7-6-48.5` Stock (size row 15) | 4 | 3 | 4 |
| tidal-rash-guard `M` Stock (row 6) | 8 | 5 | 8 |
| coldwater-4-3-chest-zip `S` Stock (row 19) | 1 | 0 | 1 |
| Orders | 0 | 3 (WL-250098, WL-604604, WL-199446) | 0 (deleted) |
| Users | 2 | 4 (checkout-test-a / b @example.com) | 2 (deleted) |
| Carts | 2 | 4 (the test users' carts) | 2 (deleted) |

**Checkpoint 2 + QA (T39, T41, T42, browser)**

| Data | Before | During | After |
|---|---|---|---|
| driftwood-7-6-mid-length `7-6-48.5` Stock | 4 | 3 | 4 |
| tidal-rash-guard `M` Stock | 8 | 6 | 8 |
| tidal-rash-guard `XL` Stock (row 8, sold-out re-check) | 8 | 0 | 8 |
| coldwater-4-3-chest-zip `S` Stock | 1 | 0 | 1 |
| Orders | 0 | 2 (WL-715240, WL-901647) | 0 (deleted) |
| Users | 2 | 4 (e2e-a-943813 / e2e-b-943813 @example.com, registered through the app's `/auth/register`) | 2 (deleted) |
| Carts | 2 | 4 (A's guest cart, re-owned at register, and B's cart) | 2 (deleted) |

Not touched: every other product, size and cart, the 2 pre-existing users and their carts, and all schemas.

## Follow-up 2026-10-11: order speed and read-only admin fields

**Speed (per step, real orders on Postgres + Resend SMTP)**

| | Before | After |
|---|---|---|
| Confirmation email (SMTP), awaited in the request | 928–1671ms warm, 7260ms first after restart | not in the request (fire-and-forget) |
| Everything else in Strapi (JWT, validation, cart lookup, transaction incl. mock charge) | 39–68ms | unchanged |
| Strapi `POST /api/orders` | 1261–2131ms | 23–52ms |
| Next `POST /api/checkout` | 1282–1659ms | 30–52ms (52ms first after Strapi restart) |
| Click Place Order → confirmation visible (browser) | — | 105–114ms warm, 212ms first visit |

The email is still sent: in a live run the send started, the controller answered 2ms later (67ms total), and SMTP finished about 6s afterwards. Resend's sandbox rejected the `@example.com` test recipient, which was logged.

**Read-only Order fields (admin, live with a temporary Super Admin)**
- **List view:** "Create new entry" is absent.
- **Edit view:** every locked field is greyed out, including PlacedAt (date and time pickers), every field of an expanded line, and the line's Add an entry / delete controls. IdempotencyKey is hidden. Status, User and ShippingAddress stay editable.
- **Status:** changed in the UI and saved; the locked values were unchanged afterwards.
- **ShippingAddress:** City and Address2 changed through the admin API and saved (200).
- **Locked fields:** changing Total or Email through the admin API → 400 "These order fields are read-only: Total." / "…: Email."
- **Create / clone:** through the admin API → 403.
- **Checkout:** still places orders with the guard on (201).

**Data changed and restored:**

| Data | During | After |
|---|---|---|
| Orders | 24 test orders (perf-* users) | deleted |
| Users | 7 perf-* @example.com users | deleted, with their carts |
| Admin users | 1 temporary Super Admin (qa-admin-temp@example.com) | deleted |
| Stock: tidal-rash-guard S / M / L / XL | 8 / 8 / 8 / 8 → 1 / 2 / 2 / 3 | 8 / 8 / 8 / 8 |
| Stock: driftwood 7-6-48.5, coldwater S | not touched | 4, 1 |

The before and after snapshots are identical: 1 order, 2 carts, 2 users. The shopper's own order WL-714224 was not touched.

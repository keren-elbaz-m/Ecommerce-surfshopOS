# Tasks — Add to Cart

> Epic: storefront-catalog
> Spec: 2026-10-10-add-to-cart
> Branch: feat/storefront-catalog/add-to-cart

## Strapi cart

- [x] T1 Add Vitest to `cms/` (devDependency, `npm test`, `vitest.config.ts`). Write failing `cms/tests/cart/cart-logic.test.ts`:
  - sizeKeyOf board/standard (`6-0-29.4`, `OneSize`); findSize.
  - addLine merge +1 and clamp; addLine on sold out → error; setLineQty clamp; removeLine.
  - mergeLines sum + clamp.
  - revalidate: adjusted; soldOut (Stock 0); unavailable (missing SizeKey, distinct from soldOut); pruned when the populated Product is null (deleted **or unpublished**).
  - Service-level test that the `Lines.Product` populate passes `status: 'published'` (strapi document service mocked).
  - toCartResponse has no stored price. (AC3–AC5)
- [x] T2 Implement `cms/src/components/cart/line.json`, `cms/src/api/cart/content-types/cart/schema.json` and `cms/src/api/cart/cart-logic.ts` (AC1, AC3, AC4)
- [x] T3 Implement `cms/src/api/cart/{routes,controllers,services}/cart.ts`:
  - custom routes with auth false
  - JWT / token identity
  - 400 / 401 / 404 / 409 mapping
  - GET creates nothing; first POST creates and returns the token
  - merge deletes the guest cart (AC2, AC5)
- [x] T4 Live check against local Strapi with curl: the full guest flow, the user flow, merge, 401 on a bad JWT, no `/api/carts` core CRUD, an edited board size → `unavailable`, and Product schema unchanged (`git diff cms/src/api/product` empty). **⏸ Checkpoint: stop for Ori's review before T5.** **2026-10-10 (guest part):** against the running `strapi develop`: GET with no cart → empty, nothing created; core `/api/carts` → 404; first add (ci-pro `5-8-24.3`, Stock 1) issued a token; re-add stayed 1 (clamped); samurai M added; PATCH 30 → 24 (Stock); sold-out `6-6-40.6` → 409; unknown size / product → 404; qty 0 → 400; DELETE removed the line; bad JWT → 401; unknown token → empty. Product schema diff empty; cms 43/43 tests, `tsc` clean. The user flow, the merge and an edited-size `unavailable` were moved to T21 (browser login after T10), per Ori; the store's user and merge paths are unit-tested in `cart-store.test.ts`

## Frontend data

- [x] T5 Failing tests:
  - `__tests__/lib/strapi/client.test.ts` (`strapiRequest`: no-store, method/body/headers, never throws, `{ok,status,data}`)
  - `__tests__/lib/strapi/cart.test.ts` (each call's path/method/headers; shape checks)
  - `__tests__/lib/strapi/sizes.test.ts` (`boardSizeKey`/`standardSizeKey`, same cases as T1)
- [x] T6 Implement `strapiRequest`, `lib/strapi/cart.ts`, and the size keys in `sizes.ts`
- [x] T7 Failing tests:
  - `__tests__/components/cart/cart.test.ts`: `toCartView` labels, prices, count, subtotal excluding sold out, remaining/progress at 0, 74, 75 and 200, flags (AC7, AC13)
  - `__tests__/lib/cart/cookie.test.ts`
  - `__tests__/lib/cart/merge-on-auth.test.ts`: merges and clears the cookie; no cookie → no call; failure is logged and never throws (AC5)
- [x] T8 Implement `components/cart/cart.ts`, `lib/cart/cookie.ts`, `lib/cart/merge-on-auth.ts`, `lib/shipping.ts`
- [x] T9 Failing tests `__tests__/app/api/cart/route.test.ts` and `lines/route.test.ts`:
  - identity headers from the cookies
  - body validation 400
  - new token sets `westline_cart` with the AC6 options
  - 409 / 404 pass through
  - Strapi down → 503
  - no-store (AC6)
- [x] T10 Implement `app/api/cart/route.ts` and `app/api/cart/lines/route.ts`. Add the one-line `withGuestCartMerge` call to the login, register, reset-password and oauth/complete routes; extend their existing tests to assert the call

## UI

- [x] T11 Add `plus` / `minus` to `Icon` SHAPES (test in the icons suite)
- [x] T12 Failing tests:
  - `CartProvider.test.tsx`: loads once; add/setQty/remove replace the cart; add success opens the drawer; per-line pending; error state (AC8)
  - `CartNavButton.test.tsx`: badge hidden at 0 and while loading; count; accessible name; toggles the drawer (AC10)
- [x] T13 Implement `cart-context.ts`, `CartProvider.tsx`, `CartNavButton.tsx`, `cart-texts.ts`. Mount `CartProvider` in `app/[locale]/layout.tsx` and swap the cart Link in `SiteNav` (site-nav tests updated). Live: badge count is correct after a curl add plus a reload. **⏸ Checkpoint: stop for Ori's review before T14 (drawer and page).** **2026-10-10:** live through Next with a cookie jar: two adds → count 2, `$158`, free shipping unlocked; `westline_cart` is HttpOnly; Chrome shows "Cart, 2 items" with badge 2, kept after reload; a new visitor has no badge. Found and fixed: `components/cart/index.ts` re-exported `useCart`, pulling `createContext` into the server layout (500 on `/api/cart`). Per Ori, the T13 checkpoint was cancelled, so this ran straight on
- [x] T14 Failing tests:
  - `ShippingProgress.test.tsx` (AC13)
  - `CartLines.test.tsx` (AC14):
    - stepper labels; − disabled at 1, + disabled at maxQty
    - Remove; pending disables the line
    - adjusted note; sold-out and unavailable states (distinct copy), struck totals, blocking note
    - removed notice; both variants
  - `CartDrawer.test.tsx` (AC11, AC15, AC16):
    - dialog, head count, closing (Esc, overlay, X, route change)
    - focus to close and return to opener; scroll lock
    - View Cart href; disabled Checkout + note; empty state
- [x] T15 Implement `ShippingProgress.tsx`, `CartLines.tsx`, `CartDrawer.tsx` (rendered by `CartProvider`)
- [x] T16 Failing tests:
  - `CartPage.test.tsx` (AC12, AC14–AC17): breadcrumb, H1, rows, summary rows, Total = subtotal, disabled Checkout + note, Continue Shopping href, sold-out blocking note, empty state, skeleton, error + Retry, none of the hidden items
  - `CartSummary.test.tsx`
- [x] T17 Implement `CartSummary.tsx`, `CartPage.tsx`, `app/[locale]/cart/page.tsx` (dynamic, noindex metadata)

## Detail pages

- [x] T18 Failing tests:
  - mapper tests (`documentId`, `sizes[].key`)
  - `SurfboardBuyPanel` / `StandardBuyPanel`: click calls `add` with documentId + the selected size key; "Adding…" disabled while pending; drawer opens on success; error note; 409 note; sold-out panels unchanged (AC9)
  - trust/shipping texts unchanged as rendered strings (AC13)
- [x] T19 Implement: mappers, panel wiring, `documentId` prop from `SurfboardDetail` / `StandardProductDetail`, `product-detail-texts.ts` additions and threshold-driven shipping strings

## Docs

- [x] T20 Confirm the Task 1 Changelog rows on the related specs still match what was built; add a Changelog row here for anything that diverged **2026-10-10:** surfboard-, standard-product-detail-page and product-content-model rows still match. Extra rows added on site-nav (site-nav AC9 "no cart count" superseded; `cartLabel` removed; SiteNav tests wrap a cart context) and account-signup-login (the locale layout reads `getSession()` for `CartProvider signedIn`)

## Verification

- [x] T21 Run the full frontend suite and the cms suite (testing skill), `npm run lint`, `npx tsc --noEmit` (frontend + cms). Live check against local Strapi: **2026-10-10:** Chrome (playwright-core), real cookies, 6 phases, every scenario passing; details in qa-report.md. Found and fixed during the run: focus return after Add to Cart (A2), cart refresh on login/logout (E3/E5), and drawer links on the current page (F5). Data changes were made with a scratchpad Document Service script and all restored (see qa-report.md)
  - add a surfboard size and a standard size → drawer opens, badge 2
  - re-add → qty 2; + stops at Stock
  - reload keeps the cart
  - set a size's Stock to 0 in admin → "Sold out" + excluded from subtotal
  - edit a board size's VolumeL in admin → "This size is no longer available." + excluded from subtotal
  - lower Stock below qty → clamped + note
  - change Price → new price shown
  - delete the product → removed notice
  - login with a guest cart → merged and cookie cleared; logout → empty guest cart
  - `/cart` empty state
- [x] T22 Run QA against acceptance criteria (qa skill), including a browser check at 1440, 900, 480 and 390 against `cart.html` and the PDP drawer (drawer width/slide/overlay, sticky summary, grid collapse, badge position, focus trap), plus a PDP regression pass (ci-pro, samurai-pro-22-boardshort) **2026-10-10:** PASS WITH NOTES, see [qa-report.md](qa-report.md)

# QA Report — Add to Cart

> Spec: spec.md
> Date: 2026-10-10 · QA by: Ori Chai Matan · Mode: setup
> Verdict: PASS WITH NOTES

## Acceptance Criteria

| AC | Criterion | Result | Evidence |
|---|---|---|---|
| AC1 | `api::cart.cart` + `cart.line` component; no core CRUD, no public permissions; Product schema unchanged | pass | `cms/src/api/cart/content-types/cart/schema.json`, `cms/src/components/cart/line.json`. Live: `GET /api/carts` → 404 (no core router). `git diff cms/src/api/product cms/src/components/product` is empty. Carts were listed and deleted through the Document Service during cleanup. |
| AC2 | Custom routes; JWT or `x-cart-token` identity; GET creates nothing; first POST issues the token; 400 / 401 / 404 | pass | `cart-store.test.ts` (9) and `cart-logic.test.ts` parseLineBody (12). Live curl (T4): GET with no cart → empty, nothing created; first add returned a token; qty 0 → 400; bad JWT → 401; unknown size / product → 404. DELETE takes its line in the path (Changelog). |
| AC3 | One line per product + size key; POST +1 clamped; PATCH clamped; sold out → 409; DELETE; key format | pass | `cart-logic.test.ts` (addLine, setLineQty, removeLine, sizeKeyOf incl. `"27.50"` → `5-10-27.5`). The frontend `sizes.test.ts` runs the same cases. Live: re-adding ci-pro `5-8-24.3` (Stock 1) stayed at 1; PATCH 30 on M (Stock 24) → 24; `6-6-40.6` → 409. |
| AC4 | Re-validated against the live published product: clamp + `adjusted`, `soldOut`, separate `unavailable`, prune deleted / unpublished with `removedCount`, no stored price | pass | `cart-logic.test.ts` revalidate (5) and toCartResponse. `cart-store.test.ts`: reads use `status: 'published'`, and a null (unpublished) product is pruned and written back. Browser (T21 B/C): M Stock 0 → "Sold out"; ci-pro Stock 4 → 2 → "Only 2 left — quantity updated." once, then gone after reload; price 600 → 650 → line total `$1,300`; VolumeL 29.9 → 30 → "This size is no longer available." (labelled `6'0" · 29.9L` from the key, not "Sold out"); deleted product → pruned with the notice. |
| AC5 | Merge on login, register, reset-password and OAuth complete; guest cookie cleared; a failed merge never fails auth; logout → fresh guest cart | pass | `merge-on-auth.test.ts` (3), plus a merge test in each of the 4 auth route suites. Browser (T21 E): register through `/api/auth/register` with a guest cart → session set, `westline_cart` cleared, cart kept (badge 1), and the old token reads empty. Logout → badge cleared. A new guest cart (samurai L) then logging in through the login page → badge 3 and `/cart` shows samurai M, ci-pro and samurai L. Reset-password and OAuth were checked by unit tests only (see Notes). |
| AC6 | `/api/cart` and `/api/cart/lines` proxy with cookie identity; cookie options; body validation; 503; no-store | pass | `route.test.ts` (3) and `lines/route.test.ts` (15). Live: the cookie is `HttpOnly`, `SameSite=Lax`, path `/`, about 30 days; the response has `Cache-Control: no-store`. |
| AC7 | `toCartView` labels, prices, totals, flags | pass | `cart.test.ts` (12): labels incl. unavailable-from-key, `$24.50` / `$49`, count over all lines, subtotal excluding blocked lines, progress at 0 / 74 / 75 / 200, rounding to cents. |
| AC8 | Provider in the layout, loads once, replaces the cart from the server, one mutation per line | pass | `CartProvider.test.tsx` (12), incl. per-line pending, failure → reload, and reload when `signedIn` changes. Live: logout and login update the badge without a full reload (T21 E3/E5). |
| AC9 | Both buy panels add the selected size; "Adding…"; error and 409 notes; sold-out unchanged; `documentId` and size keys in the mappers | pass | `SurfboardBuyPanel.test.tsx` and `StandardBuyPanel.test.tsx` (3 new each), and mapper tests (`documentId`, `key`, missing documentId → null). Live: ci-pro `6'0" · 29.9L` and samurai M added from the PDPs; the sold-out panels were unchanged in the regression pass. |
| AC10 | Header button: badge styling, "Cart, N items", hidden at 0 / loading, toggles the drawer, not on /cart | pass | `CartNavButton.test.tsx` (5) and `isCartPath` tests. Browser: badge is 16×16, 4px right and 2px above the bag at every width (design `left:-14px; top:-8px` on a 20px icon); the header button does nothing on `/cart` (T21 F2). |
| AC11 | Drawer: dialog, 420px / full width ≤480, overlay, closing paths, scroll lock, focus in / trap / return, contents | pass | `CartDrawer.test.tsx` (11). Browser: x1020 / w420 at 1440, identical to the design's `.cart-drawer`; full width at 480 and 390. Esc, overlay, X and navigation close it; body overflow is `hidden`; Tab stays inside; focus returns to Add to Cart (fixed in T21, see Issues). |
| AC12 | `/cart`: thin dynamic route, noindex; breadcrumb, H1; `1.7fr 1fr` / 48px, one column ≤900; rows; sticky summary; skeleton; error + Retry | pass | `CartPage.test.tsx` (7), `CartSummary.test.tsx` (4). Browser vs `cart.html`: columns `720.281px 423.719px` and gap 48px at 1440, one column of 852 / 448 / 358px at 900 / 480 / 390, all identical to the design. Photos are 110×138; H1 is Poppins at 42 / 36 / 28 / 28px, identical. No horizontal scroll. |
| AC13 | `FREE_SHIPPING_THRESHOLD = 75`; messages; progress bar | pass | `ShippingProgress.test.tsx` (2), `cart.test.ts`. PDP trust and accordion texts still render "Free shipping over $75" / "…orders over $75." (existing tests unchanged). Live: "You've unlocked free shipping!" at $2,570 and "Add $75 more for free shipping." at $0. |
| AC14 | Stepper limits, Remove, pending, adjusted / sold-out / unavailable copy, struck totals, blocking note, removed notice, live price | pass | `CartLines.test.tsx` (12). Browser T21 A5 (+ disabled at Stock 4), B, C and D (− disabled at 1, Remove → subtotal `$79`). |
| AC15 | Checkout disabled with "Checkout is coming soon."; no checkout route | pass | Drawer, summary and page tests. Live: disabled in both places. No `/checkout` route exists. |
| AC16 | Empty state in the drawer and page; page hides the summary and shipping block | pass | Tests, plus T21 F1 / F3. |
| AC17 | No promo code, recommendations, notes, wishlist, tax line, shipping estimate or PDP qty picker | pass | `CartPage.test.tsx` "shows none of the hidden design items"; no inputs or textareas on the page. |

## Checks Run

- tests (frontend): `npx vitest run` passes: 473 passed, 0 failed, 54 files (352 / 40 before this spec).
- tests (cms): `npm test` (Vitest, new) passes: 43 passed, 2 files.
- lint: `npm run lint` passes with 1 pre-existing warning (`app/[locale]/layout.tsx` no-page-custom-font).
- typecheck: `npx tsc --noEmit` is clean in `frontend/` and in `cms/`.
- build: `next build` was **not run**: your `next dev` on :3000 shares `.next`. A one-off Tailwind build confirms every new class compiles (`bg-surface`, `animate-drawer-in`, `animate-fade-in`, `grid-cols-[1.7fr_1fr]`, `max-[480px]:w-full`, `[&>*]:w-full`, …).
- browser: playwright-core with installed Chrome against your running `strapi develop` and `next dev`. T21 (6 phases, real cookies) and T22 at 1440 / 900 / 480 / 390 against `westline-site-desing/cart.html` and the samurai PDP drawer. PDP regression on ci-pro, mikey-february-s-shorty, samurai-pro-22-boardshort and coldwater-4-3-chest-zip at 1440 and 390: all 200, Add to Cart present, trust text intact, no horizontal scroll, no page errors. Screenshots are in the session scratchpad, not committed.

## Issues Found

Found and fixed during verification, each with a failing test first:

1. **Server/client boundary (T13):** `components/cart/index.ts` re-exported `useCart`, pulling `createContext` into the server layout, so `/api/cart` returned 500. The barrel now exports only the provider and the button.
2. **Line refs (T15):** `CartLines` passed the whole line view to `setQty` / `remove`, so the request body would have carried every view field. It now passes `{ productDocumentId, sizeKey }`.
3. **Focus return (T21 A2):** the Add to Cart button is disabled while pending, so it lost focus and Esc returned focus to `<body>`. `add()` now records the opener when it starts.
4. **Session change (T21 prep):** after logout or login, the client provider kept the previous session's cart until a full reload. The layout now passes `signedIn` and the provider reloads.
5. **Drawer links (T21 F5):** in the drawer, Continue Shopping on `/products` didn't close the drawer (same pathname). The links now close it.
6. **Design fidelity (T22):** the free-shipping banner and photo frames used `bg-background` (white) where the design uses `--surface-2`. A `surface` token (`#F9F9F9`) was added.

## Notes

1. **Restart `next dev` to see the new Tailwind token and animations.** `tailwind.config.ts` gained `surface`, `drawer-in` and `fade-in`. The dev server running since before this change won't pick them up. Until a restart, the banner and photo frames render white and the drawer appears without sliding in; the T22 screenshots were taken before the token existed. This is the same situation as the `danger` token on standard-product-detail-page.
2. **Test data changes, all restored.** Made through the Document Service with a scratchpad script, against the local DB only, after snapshotting the originals:
   - **samurai-pro-22-boardshort:** M Stock 24 → 0 → restored to 24.
   - **ci-pro:** Price 600 → 650 → restored to 600.
   - **ci-pro, 6'0" size:** Stock 4 → 2, then VolumeL 29.9 → 30 → restored to 29.9 / Stock 4.
   - Restoring rewrote the full size lists in their original order; the after/before comparison by value is identical.
   - **"QA Temp Cart Product"** (`qa-temp-cart-product`) was created for the deleted-product case (reusing samurai's first image and the Clothing category), then deleted. Strapi no longer returns it. Next's 60-second fetch cache kept its page at 200 briefly afterwards; that's existing behaviour.
3. **Left in your local DB on purpose:** the throwaway user `qa-cart-1791659349588@example.com` (password `QaCart-2026-test`) and its now-empty cart, so you can sign in with it. All guest test carts (3) were deleted.
4. **Emails sent:** registering the throwaway user went through `/api/auth/register`, which sends the welcome email via your SMTP to that `example.com` address.
5. **Not verified live:** the merge after reset-password and after Google OAuth. Both are the same one-line `withGuestCartMerge` call as login and register and are covered by unit tests; a live check needs a reset email link or a Google login.
6. **Summary at ≤900px is static** (the design keeps it sticky in the single column). This matches the PDP buy column and avoids the summary overlaying lines.
7. **Tech debt carried to the checkout specs** (spec Out of Scope): abandoned guest carts are never cleaned up, and cart creation isn't rate-limited.
8. **No PR yet.** The branch `feat/storefront-catalog/add-to-cart` is local and uncommitted, at your request.

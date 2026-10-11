# QA Report — Standard Product Detail Page

> Spec: spec.md
> Date: 2026-10-07 · QA by: Ori Chai Matan · Mode: setup
> Verdict: PASS WITH NOTES

## Acceptance Criteria

| AC | Criterion | Result | Evidence |
|---|---|---|---|
| AC1 | Standard → `StandardProductDetail`, Surfboard → `SurfboardDetail`, one fetch; surfboard tests unchanged except imports | pass | `ProductDetailPage.test.tsx`: "renders a Standard product from a single fetch" (`getProductBySlug` called once; no meters, no Dimensions). `ProductDetailPage.tsx` switches on `SizeType`. The surfboard tests (`Surfboard*.test.tsx`, `surfboard-detail.test.ts`) differ from `HEAD` in import paths only, and "404s for a non-surfboard product" was replaced by "404s for a product type no layout renders". Live: samurai, coldwater, the cap and the tee render the standard layout; ci-pro and mikey render the surfboard layout. |
| AC2 | Query adds `fields[5]=Gender` and `populate[StandardSizes]=true`; still one request, revalidate 60; no Strapi change | pass | `product.test.ts`: "also requests Gender and StandardSizes…" and "passes a standard product through…". `lib/strapi/product.ts` `productDetailQuery`. `cms/` is untouched (git status). |
| AC3 | 404 for an unknown slug, an unknown SizeType, or a Standard product missing a required field | pass | `standard-detail.test.ts` (9 null cases). `ProductDetailPage.test.tsx`: "404s for a product type no layout renders" and "404s when a required field is missing (AC3)". Live: `/products/nope` and `/products/category` return 404. |
| AC4 | Unavailable → 200, Home breadcrumb, message | pass | `ProductDetailPage.test.tsx` "shows the unavailable state…" (unchanged, now rendered by `shared/ProductUnavailable.tsx`). |
| AC5 | Breadcrumb with a gender level for Men/Women clothing only | pass | `breadcrumb.test.ts`, `standard-detail.test.ts` (gender), and the page test "shows the gender level…". Live: samurai shows `HOME / CLOTHING / MEN / SAMURAI PRO 22" BOARDSHORT` and Men links to `?gender=men`. archive-team-tee (Unisex) shows `HOME / CLOTHING / ARCHIVE TEAM TEE`. coldwater shows `HOME / WETSUITS / …` and the cap shows `HOME / ACCESSORIES / …`. |
| AC6 | 1240 container; `1.6fr 1fr`, gap 56; ≤900 one column, gap 28, gallery first; buy column sticky at 88px above 900, static at ≤900 | pass | Browser: the grid matches the design at every width. **1440:** `699.062px 436.938px`, gap 56px, gallery x124 / buy x879.1, identical to the design. **900 / 420 / 390:** a single column of 852 / 388 / 358px with a 28px gap, the gallery above the buy column, same as the design. The buy column is `sticky`/`88px` at 1440 and `static` at ≤900. Sticky behaviour matches the design while scrolling: it pins at 88px at scrollY 60–180. No horizontal scroll at any width. |
| AC7 | Desktop 2-up grid, 3px gap, fixed 4:5 frame on `bg-background`, `object-contain`, priority first, single photo spans both columns | pass | Browser at 1440: frames are 348×435 at x124 / x475 (3px gap), the same box as the design's `.gallery-grid-item`. The frame background is the page background (white) and images use `object-fit: contain`. The first image has no `loading=lazy`; the rest are lazy. Cap and tee (one photo each): the frame is 699px wide with `col-span-2`. `StandardGallery.test.tsx` (4 grid tests). |
| AC8 | ≤900: scroll-snap x mandatory, full-width items snapping to centre with `always`, scrollbar hidden, no arrows/dots/counter | pass | Browser at 900 / 420 / 390: the gallery is `display:flex`, `overflow-x:auto`, `scroll-snap-type: x mandatory`, `scrollbar-width: none`. Items are full width (852 / 388 / 358px) with `snap-align: center` and `snap-stop: always`, all in one row. Programmatic scroll to 1.3× and 2.8× the width settles on 1× and 2× (whole photos). A real CDP touch swipe moves exactly one photo (scrollLeft = item width). No arrows, dots or counter. |
| AC9 | Shared lightbox; opens at the clicked photo; focus returns to it | pass | Browser at all 4 widths: clicking photo 3 (desktop) or the swiped-to photo 2 (mobile) opens "Photo viewer" at `3 / 5` / `2 / 5` with that photo's src. Focus moves to "Close photo viewer" and body overflow is `hidden`. Three ArrowRight presses wrap (`3 → 1`, `2 → 5`). The stage box is identical for every photo. Escape closes it, overflow is cleared, and focus returns to "Open photo 3 of 5" / "Open photo 2 of 5". A backdrop click also closes it. `PhotoLightbox.test.tsx` (8) and `StandardGallery.test.tsx` (4 lightbox tests). |
| AC10 | Eyebrow; H1 Poppins 800 uppercase `clamp(26px,3.4vw,38px)`; price 24/700, tabular, no underline | pass | Browser: the eyebrow "CLOTHING" is horizon `rgb(21,94,239)`. The H1 is Poppins / 800 / uppercase at 38 / 30.6 / 26 / 26px across 1440 / 900 / 420 / 390, identical to the design's H1 at each width. The price is 24px / 700 / `tabular-nums` with a 0px bottom border. |
| AC11 | Size grid: order, label "Size — x", 5 columns (4 at ≤420), selected ink + `aria-pressed`, default first in stock | pass | Browser: samurai shows S M L XL with S pressed and the label "SIZE — S". Clicking M changes the label to "SIZE — M" with only M pressed. The grid is 5 columns at 1440 / 900 and 4 at 420 / 390, matching the design. The cap shows "One Size". `StandardBuyPanel.test.tsx` (3 size tests). |
| AC12 | Sold out (disabled, line-through, name) / low stock (danger flag + note) / all sold out | pass | **Low stock**, browser after the dev-server restart, at 1440 and 390: coldwater S is "S — Low stock", selected by default. Its 15×15 "!" flag is `rgb(192,57,43)` (`#C0392B`, the `danger` token) at the box's top-right. The "LOW STOCK" note is `rgb(192,57,43)`, 11.5px / 600 / uppercase. The note hides when M is selected and returns for S. The tee's XL is flagged. **Sold out and all sold out:** `StandardBuyPanel.test.tsx` (disabled, `line-through`, name "S — Sold out"; CTA "Sold out" disabled; no grid without sizes). See Note 2. |
| AC13 | Add to Cart is ButtonCTA md, inert | pass | Browser: "ADD TO CART", `type=button`, enabled, no handler. `StandardBuyPanel.test.tsx` "is an inert, enabled Add to Cart button…". |
| AC14 | Shared trust list, surfboard copy | pass | Browser: samurai, ci-pro and mikey all show "Free shipping over $75 / 30-day returns / Ships in 3–5 business days" from `shared/TrustList.tsx`. |
| AC15 | Accordions: Description open by default, Shipping & Returns, one at a time, aria, chevron 180° | pass | Browser at all 4 widths: Description starts `aria-expanded=true` with its panel visible and chevron `matrix(-1,0,0,-1,…)` (180°). Clicking Shipping & Returns opens it and closes Description; clicking it again closes both. The header is 15px / 700 with 20px padding. `StandardAccordions.test.tsx` (5), including markdown rendered without raw HTML. |
| AC16 | No swatch, Size Chart, pill, wishlist, related, quantity, Details & Features accordion, See details, cart drawer | pass | `ProductDetailPage.test.tsx` "renders none of the hidden design features (AC16)". Screenshots show none of them. (The only heart is the site nav's wishlist link, which is not on the page.) |

### Surfboard regression (ci-pro, mikey-february-s-shorty at 1440 and 390)

- **Columns:**
  - 1440: `330px 394px 380px`, boxes at L124 / 498 / 936 and W330 / 394 / 380. These are identical to the surfboard QA of 2026-10-06.
  - 390: stacked gallery (y128) → buy (y826) → description (y1177).
- **Attribute meters:** 12 per board. Max fill and marker error is 0.0042% (0 at 1440 on ci-pro).
- **Dimensions select:** ci-pro has 10 options with "6'6" · 40.6L — Sold out" disabled. Mikey has 5 options with "6'4" · 32.9L — Sold out" disabled.
- **Gallery frame:** stays fixed while flipping through every photo. The gallery is sticky at 1440 and static at 390.
- **Lightbox (now the shared `PhotoLightbox`):** opens with focus on close and scroll locked. The arrow keys step it (`2 / 4`, `2 / 2`). Escape closes it and returns focus to "Open photo 2 of …", so the main image stayed in sync.
- **Horizontal scroll:** none.

## Checks Run

- build: `next build` was **not run**. Your `next dev` on :3000 shares `.next`, the same reason as the surfboard QA. `tsc` plus the dev server rendering every route stand in for it.
- lint: `npm run lint` passes with 1 pre-existing warning (`app/[locale]/layout.tsx:30` no-page-custom-font, untouched).
- typecheck: `npx tsc --noEmit` passes.
- tests: `npx vitest run` passes: 352 passed, 0 failed, 40 files. There were 287 before this spec, all still passing.
- browser: playwright-core (npx cache) with installed Chrome, at 1440 / 900 / 420 / 390, against `westline-site-desing/product-apparel-samurai-boardshort.html` rendered at the same widths. Screenshots are in the session scratchpad and not committed.

## Issues Found

(none)

## Notes

1. **Dev server restart was needed for the `danger` token.** On the first browser pass, the low-stock flag rendered transparent and the note in the default text colour. The `next dev` serving :3000 had been started on 2026-10-06 and never loaded the new `danger` colour from `tailwind.config.ts`. After a restart (pid 10641, 2026-10-07 21:19), both are `#C0392B`. This was environment, not code: a one-off Tailwind build had already shown `bg-danger` / `text-danger` compile.
2. **Sold-out size styling is verified by tests only.** No seeded standard product has a size at Stock 0, so the disabled / line-through state wasn't seen in a browser. The surfboard regression covers sold-out options on that page.
3. **Samurai has 5 images in Strapi**, while `cms/seed/catalog/samurai-pro-22-boardshort/` has 6 files and the design shows 6. The page renders whatever Strapi returns; this is a content/seed matter, not this spec.
4. **The sticky buy column only pins briefly.** It is nearly as tall as the gallery (1120px vs a 1311px grid with 5 photos), so it releases after about 190px of scroll. The design has the same behaviour (1097px column) and releases about 23px later.
5. **Layout shift is ≈0.** Frame boxes with every image request blocked are identical to the loaded ones at all 4 widths. Measured CLS is ≤0.00026, from the web-font swap, not the gallery.
6. **No PR yet.** The branch `feat/storefront-catalog/standard-product-detail-page` is local and uncommitted, at your request. `gh` isn't installed here, so open the PR with the pr skill once committed.

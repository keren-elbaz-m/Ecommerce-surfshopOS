# Tasks — Product Card

> Epic: storefront-catalog
> Spec: 2026-09-28-product-card
> Branch: feat/storefront-catalog/product-card

## Data

- [x] T1 Write failing tests `frontend/src/__tests__/lib/strapi/media.test.ts` (relative → absolute, absolute passthrough) and `frontend/src/__tests__/shared/components/product-card/product-card.test.ts` (AC1–AC3, `formatPrice` whole/fractional, string price `"829.00"` → 829 / `$829`, non-numeric price → null, ≤4 images, null on missing Name/Slug, null on empty/missing Images, fit contain/cover/missing Category)
- [x] T2 Implement `frontend/src/lib/strapi/media.ts` (`strapiMediaUrl`) to pass T1; switch `frontend/src/lib/strapi/homepage.ts` to it and delete its local `absoluteUrl` (existing `homepage.test.ts` + hero tests stay green). **2026-09-28:** homepage.test.ts 7/7 and hero-carousel 14/14 green after the switch
- [x] T3 Implement `frontend/src/shared/components/product-card/product-card.ts` (`ProductCard`, `StrapiProduct`, `toProductCard`, `formatPrice`) to pass T1

## Component

- [x] T4 Write failing tests `frontend/src/__tests__/shared/components/product-card/ProductCard.test.tsx`: h3 + formatted price; overlay link href + "View <name>"; first alt = name, rest ""; N dots at 2–4, capped at 4; dot click → aria-current + active photo; object-contain vs object-cover; no dots with 1 image; heart absent unless both props; aria-pressed + label per isFavorite; click calls onToggleFavorite(slug)
- [x] T5 Implement `ProductCardMedia.tsx` (client: photo stack, dots, heart) in `frontend/src/shared/components/product-card/`
- [x] T6 Implement `ProductCard.tsx` (shell: article, overlay Link, media, body) + `index.ts` barrel to pass T4. Also added `./src/shared/**` to `frontend/tailwind.config.ts` `content`: it only scanned pages/components/app, so every class under `src/shared/` would have been missing from the CSS

## Verification

- [~] T7 Run the full frontend suite + coverage on changed files (testing skill); `npm run lint` and `tsc --noEmit` pass. **2026-09-28:** 75/76 pass; the one failure is pre-existing and outside this spec (`hero.test.tsx` AC13 expects "Coming Soon" but home-hero's fallback says "Something Wrong!" since 55de6d6, and it fails on 09caffb too); lint clean (pre-existing font warning), `tsc` clean. **Open:** coverage — no `@vitest/coverage-v8` installed
- [ ] T8 Run QA against acceptance criteria (qa skill) — visual comparison against the design happens here via a test render; the first real page render comes with catalog-listing-page

## Tidy-up (frontend/components standard)

- [x] T9 Align with frontend/components: `productHref` from `lib/routes.ts`, `STRAPI_URL` from `lib/strapi/client.ts`, `HeartIcon` → shared `Icon`. 2026-09-29: card + mapper tests green; heart SVG output identical
- [x] T10 Add `product-card-texts.ts` and use it in ProductCard and ProductCardMedia. 2026-09-29: card tests green with unchanged literal assertions
- [x] T11 Pin the price to the bottom: flex-column article + flex-1 body + `mt-auto` price row in `ProductCard.tsx`; test the classes in `ProductCard.test.tsx`; full suite green. 2026-09-29: 181/181; live `/products` + `/products/category/clothing` HTML differs only in the card's three class strings (article, body, price row); `/` byte-identical

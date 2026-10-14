# QA Report — Home Featured Gear

> Spec: spec.md
> Date: 2026-10-14 · QA by: Ori Chai Matan · Mode: setup
> Verdict: PASS WITH NOTES

## Acceptance Criteria

| AC | Criterion | Result | Evidence |
|---|---|---|---|
| AC1 | Section below the hero; copy from the Homepage with fallback; Featured products newest first, up to 8, server-rendered | pass | **Runtime:** "New in" / "Featured gear" (fallback, since the local Homepage predates the fields) with the 6 seeded featured products, newest first: Mikey February's Shorty, Two Happy, CI 2.Pro, Samurai Pro 22" Boardshort, Soft Cruiser 8'0" Longboard, Twin Fin 5'8" Keel. **Tests:** `FeaturedGear.test.tsx` (Homepage copy, fallback ×2), `products.test.ts` (Featured filter, `updatedAt:desc`, 8), `homepage.test.ts`, Home route test (Hero then FeaturedGear), `seed-homepage.test.ts` (fields seeded) |
| AC2 | Shared card with the wishlist heart | pass | **Runtime (signed in):** first card's heart "Add to wishlist" → pressed; `/api/wishlist` → `["mikey-february-s-shorty"]`. **Test:** `FeaturedSlider.test.tsx` |
| AC3 | Slider: 4 / 2 / 1 per view, arrows at the edges, dots per position | pass | **Runtime:** at 1280, 4 visible, prev hidden, next shown, 3 dots; Next → both arrows and current dot 2; dot 3 → next hidden. At 900, 2 visible and 5 dots. At 375, 1 visible and 6 dots. **Tests:** `stepCount`, arrows and dots in `FeaturedSlider.test.tsx` |
| AC4 | No section without products or with Strapi down | pass | **Test:** `FeaturedGear` returns null for `[]` and `null` |
| AC5 | Layout 1280/900/375, no horizontal scroll | pass | **Runtime:** `scrollWidth > innerWidth` false at all three; background `rgb(249,249,249)`; screenshots checked against the design |

## Checks Run

- frontend: vitest 99 files / 890 tests, `tsc`, lint, build — pass
- cms: `tsc`, 22 files / 310 tests — pass

## Issues Found

(none)

## Notes

- **Local Homepage:** it was created before `FeaturedEyebrow`/`FeaturedTitle` existed (the seed is create-only), so it shows the fallback copy. Set the fields in Content Manager → Homepage to change it.
- **Desktop arrows:** these, when more than 4 products are featured, are Ori's extension of the design's static 4-column grid. With 4 or fewer, desktop shows the plain row.
- **Throwaway user:** one, and its wishlist row, deleted afterwards.

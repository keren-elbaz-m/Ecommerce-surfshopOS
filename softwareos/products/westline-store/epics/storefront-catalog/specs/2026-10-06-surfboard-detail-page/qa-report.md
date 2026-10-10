# QA Report — Surfboard Detail Page

> Spec: spec.md
> Date: 2026-10-06 · QA by: Ori Chai Matan · Mode: setup
> Verdict: PASS WITH NOTES

## Acceptance Criteria

| AC | Criterion | Result | Evidence |
|---|---|---|---|
| AC1 | `/products/<slug>` renders surfboards; route is a one-line wrapper; catalog routes unaffected | pass | `app/[locale]/products/[productSlug]/page.tsx` (one line); live: tideline and ci-pro 200; `/products` and `/products/category/surfboards` 200 |
| AC2 | One `strapiFetch` request by Slug with all fields and populates, revalidate 60 | pass | `product.test.ts` "requests one product by slug…"; `lib/strapi/product.ts` `productDetailQuery`; the slug is URL-encoded (test "encodes the slug…") |
| AC3 | 404 for an unknown slug, `category`, non-surfboard, or a missing required field | pass | `ProductDetailPage.test.tsx` "not found" (3 tests); `surfboard-detail.test.ts` null cases (8); live: `/products/nope`, `/products/category` and `/products/samurai-pro-22-boardshort` 404 |
| AC4 | Strapi down → 200, Home breadcrumb, "This product couldn't be loaded right now." | pass | `ProductDetailPage.test.tsx` "shows the unavailable state…"; `product.test.ts` network / 500 / malformed → `unavailable` |
| AC5 | Breadcrumb Home / Category / Product; eyebrow; H1 Poppins 800 `clamp(24px,2.1vw,30px)`; 64×2 rule | pass | Tests "renders the breadcrumb, eyebrow and heading"; browser: breadcrumb is JetBrains Mono 12px uppercase; H1 is Poppins 800 uppercase, 30px at 1440 and 24px at 1100 / 900 / 390 |
| AC6 | 330/1fr/380 + 44px gap; 280/1fr/340 at ≤1100; single column gallery → buy → desc at ≤900; sticky top 96px | pass | Browser column boxes match the design's `.pdp-grid3` at every width: 1440 L124/498/936 W330/394/380; 1100 L24/334/736 W280/372/340; 900 and 390 stacked image → buy → desc. Gallery and buy are `sticky` / `96px` above 900 and `static` at ≤900. No horizontal scroll at 390 |
| AC7 | Main image with wrapping prev/next; priority first image; no arrows for 1 image; opens the lightbox | pass | `SurfboardGallery.test.tsx` (3 tests); screenshot `top-app-1440` |
| AC8 | Lightbox dialog: close / backdrop / Esc, arrow keys wrap, scroll lock, focus to close and back | pass | `SurfboardGallery.test.tsx` (7 tests); browser at all 4 widths: opened, `body.overflow=hidden`, focus on "Close photo viewer", Esc closed it, focus returned to "Open photo 1 of 2" |
| AC9 | Video when set; markdown Description (no raw HTML), 6-line clamp, Read more / less with `aria-expanded` only on overflow | pass | `SurfboardDescription.test.tsx` (4 tests); `ProductDetailPage.test.tsx` video on/off; browser: tideline clamps at 6 lines with READ MORE. No seeded board has a video, so the video block is verified by test only |
| AC10 | 12 scales in exact proportion to value: fill = value %, marker centre = value %, labels at 0 / 50 / 100 %, meter 1–100, Skill enum → 1/50/100 | pass | Unit tests: 34/1/100 exact, clamp 0 → 1 and 150 → 100, Skill mapping. Page tests: `width` / `left` / `aria-valuenow` for 34, 1, 100 and label positions. **Browser**, measured on rendered geometry for 2 boards × 4 widths (96 meters): max marker-centre error 0.004 %, max fill error 0.004 %, label anchors 0.00 px off their 0/50/100 % points, no label overlap. Live values match Strapi for all 24 meters on tideline and ci-pro |
| AC11 | Dimensions select sorted, sold-out disabled with suffix, first in-stock default; price; inert Add to Cart; sold-out state; trust list | pass | `SurfboardBuyPanel.test.tsx` (7 tests); live ci-pro: 10 options in length order, `6'6" · 40.6L — Sold out` disabled; tideline: 4 options, default 5'10" (Stock 3) |
| AC12 | Hidden: fin setup, Material / Fin System, More options, Wishlist / Share, You might also like, signature, Subtitle, cart drawer | pass | `ProductDetailPage.test.tsx` "renders none of the hidden design features" and `surfboard-detail.test.ts` "never exposes fin data"; browser text scan at all widths found no fin, wishlist, share, material or related text (see Note 1 on "House Shaper") |

## Checks Run

- build: `next build`. **Not run**: the user's `next dev` server is serving this working tree, and `next build` would overwrite its `.next` folder. `tsc` plus the dev server rendering every route stand in for it.
- lint: `npm run lint` passes (1 pre-existing warning, `app/[locale]/layout.tsx:30` no-page-custom-font, an untouched file).
- typecheck: `npx tsc --noEmit` passes.
- tests: `npx vitest run` passes (270 passed, 0 failed, 34 files). Spec-scoped: 79 passed.
- browser: Playwright (playwright-core run outside the repo, against installed Chrome) at 1440, 1100, 900 and 390, comparing against `westline-site-desing/product-surfboard-tideline.html`. Screenshots were captured in the session scratchpad and are not committed.

## Issues Found

| # | Severity | Issue | Route |
|---|---|---|---|
| 1 | low | Scale rows are ~14px taller than the design. The label row is fixed at two lines (`h-7`) so wrapped labels ("Vertical / Pocket", "Cruisy / Positional") never overlap the next item | Accept, or /spec-changes to size the row to its content |
| 2 | low | Page background is white; the design uses `#F9F9F9`. It is site-wide: the catalog body is also white | Out of this spec's scope; note for a global styling change |
| 3 | low | Trust list items are left-aligned in the panel; the design indents them slightly | Accept, or /hotfix if pixel parity is wanted |

## Notes

1. **"House Shaper" text on tideline.** It is the editor-written last line of that product's Description markdown in Strapi (`> — House Shaper, WESTLINE`), rendered as CMS content. The page has no hardcoded signature, so AC12 holds. Remove it from the content if it shouldn't show.
2. **Price format.** The page shows `$829` (shared `formatPrice`, whole numbers without decimals, as on the product card) where the design shows `$829.00`. This follows the spec (AC11 uses `formatPrice`).
3. **Video.** There is no seeded Video, so AC9's video branch is verified by tests only. Upload one in Strapi to see it live.
4. **Hygiene.** T1–T10 are `[x]`; T11 is this run. Spec `> Status: in-progress` matches (not yet merged). No `hotfix` Changelog rows. No PR yet. The work is uncommitted on `feat/storefront-catalog/surfboard-detail-page`, and that branch also carries the uncommitted catalog-listing work.

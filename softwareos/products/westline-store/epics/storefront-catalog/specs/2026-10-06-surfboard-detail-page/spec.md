# Surfboard Detail Page

> Epic: ../../epic.md
> Status: in-progress
> Branch: feat/storefront-catalog/surfboard-detail-page
> Covers: /products/*, frontend/src/app/[locale]/products/[productSlug]/**, frontend/src/components/product-detail/**, frontend/src/lib/strapi/product.ts, frontend/src/__tests__/components/product-detail/**, frontend/src/__tests__/lib/strapi/product.test.ts

## Overview

A server-rendered product detail page at `/products/<productSlug>`, the URL every ProductCard already links to. It is for **surfboards** (`SizeType = Surfboard`). Other products (`SizeType = Standard`) are rendered by [standard-product-detail-page](../2026-10-07-standard-product-detail-page/spec.md). This is the tech plan's `product-detail-page` candidate, split by product type. It is the "open a product detail page" step of the brief's success path.

The page has three columns:
- **Description:** the category eyebrow, the H1, the shaper video, the Description with Read more, and 12 attribute scales.
- **Gallery:** prev/next with a lightbox.
- **Buy panel:** a Dimensions select, the price, Add to Cart and the trust list.

Each attribute scale is drawn in exact proportion to its 1–100 value. Add to Cart is rendered but inert. add-to-cart wires it. The scales serve the mission's beginner-to-intermediate surfer, who wants to judge a board without reading raw specs.

Design source: `westline-site-desing/product-surfboard-tideline.html`. It is matched as-is except for the following:
- **Hidden items (AC12),** including the design's fin setup block.
- **Scale label positioning:** labels sit at their true points (AC10).
- **Gaps the design leaves open, decided in shaping:**
  - sold-out sizes
  - the Skill Level enum
  - an unavailable state
  - lightbox focus handling

## Goals & User Stories

- As a shopper, I open a surfboard from the catalog and see its photos, price, sizes and what it's built for, so I can judge whether it fits me.
- As a shopper, I flip through the photos and open them full-screen.
- As a shopper, I pick a length/volume and see straight away which sizes are sold out.
- As a less experienced surfer, I read the wave, performance and shape scales, each showing the board's exact value, to understand the board without decoding technical specs.
- As a content editor, a new surfboard in Strapi gets a working page without a deploy. An uploaded shaper video appears on it, and changing an attribute value moves its marker to exactly that point.

## Acceptance Criteria

1. AC1: Route.
   - `/products/<productSlug>` is server-rendered by `app/[locale]/products/[productSlug]/page.tsx`. That file is a one-line wrapper.
   - It renders a product whose Slug matches and whose `SizeType` is `Surfboard`.
   - All URLs come from `lib/routes.ts` (`HOME_HREF`, `categoryHref`).
   - `/products` and `/products/category/<slug>` keep resolving to the catalog.
2. AC2: Data comes from Strapi through `strapiFetch` (`revalidate` 60), in one request filtered by `Slug`. It fetches:
   - Name, Price, Description, SizeType
   - Images (url, width, height)
   - Category (Name, Slug)
   - BoardSizes
   - SurfboardSpecs, including Video (url, mime)
3. AC3: 404 (`notFound()`) in each of these cases:
   - No product has the slug. This includes the reserved slug `category`.
   - The product's `SizeType` isn't `Surfboard`.
   - The product lacks a required field: Name, a finite Price, at least one image, or Category.
4. AC4: Strapi unavailable (network error, non-2xx or malformed body):
   - The page renders with status 200.
   - It shows the breadcrumb (Home) and the message "This product couldn't be loaded right now."
   - It never 404s because of an outage.
5. AC5: Header and breadcrumb, per design:
   - The breadcrumb is `Home / <Category Name> / <Product Name>`. Home and the category are links, to `HOME_HREF` and `categoryHref(slug)`. The product name is plain ink text.
   - The breadcrumb uses JetBrains Mono, 12px, uppercase, muted, with a `/` separator at 50% opacity.
   - The left column starts with the Category Name as a horizon-blue eyebrow.
   - Then comes the H1: Poppins 800, uppercase, `clamp(24px,2.1vw,30px)`.
   - Then a 64×2px ink rule.
6. AC6: Layout, per design:
   - The container is 1240px max with 24px padding (16px at ≤480px).
   - Columns are `330px 1fr 380px` with a 44px gap. At ≤1100px they are `280px 1fr 340px` with a 30px gap.
   - At ≤900px the page is a single column ordered gallery → buy panel → description, with a 32px gap.
   - Above 900px the gallery and the buy panel are sticky at `top: 96px`.
7. AC7: Gallery:
   - It shows one main image in a fixed frame (column width × `74vh`, the page background token `bg-background`, never sized from the photo), centred with `object-contain`, between two 36px round prev/next buttons. Switching photos moves or resizes nothing: frame, image centre, prev/next, buy panel and description stay in place. The buttons have aria-labels "Previous photo" / "Next photo".
   - Navigation wraps around.
   - The first image loads with `priority`.
   - With a single image the prev/next buttons are not rendered.
   - The main image is a button labelled "Open photo <n> of <N>", and it opens the lightbox.
8. AC8: Lightbox:
   - It is a `role="dialog"` with `aria-modal="true"` and the aria-label "Photo viewer".
   - It shows the current photo on a dark backdrop in a fixed stage (`min(88vw,860px)` × `min(78vh,860px)`, photo `object-contain` on `bg-background`), with a close button, prev/next arrows and a "<n> / <N>" counter that don't move between photos.
   - It closes on the close button, a backdrop click, or `Escape`.
   - ArrowLeft and ArrowRight navigate and wrap. The main image stays in sync.
   - Body scroll is locked while it is open.
   - Focus moves to the close button on open and returns to the main image on close.
   - The arrows and the counter are hidden when there is a single image.
9. AC9: Description column:
   - **Video:** when `SurfboardSpecs.Video` is set, a 16:9 `<video controls playsinline preload="metadata">` appears below the rule. Otherwise there is no video block.
   - **Description:**
     - "From the shaper" is a horizon eyebrow.
     - The Description markdown is rendered with `react-markdown`, without raw HTML. It is 14px, line-height 1.7 and muted, and clamped to 6 lines.
     - A "Read more" / "Read less" toggle with `aria-expanded` and a chevron that rotates 90° expands it.
     - The toggle only renders when the text overflows 6 lines.
10. AC10: Attributes. Shown when SurfboardSpecs is present (hidden entirely otherwise): an "Attributes" eyebrow, then three sections with Poppins headings, 12 scales in total:
    - **Wave:**
      - Size (WaveSize: Knee / Double+)
      - Break (Point / Reef / Beachbreak)
      - Power (Weak / Mushy · Medium / Steep · Strong / Barrels)
    - **Performance:**
      - Approach (Vertical / Pocket · Power / Carving · Cruisy / Positional)
      - Skill Level (Beginner / Intermediate / Advanced)
      - Foot Orientation (Back Foot / Neutral / Front Foot)
    - **Shape:**
      - Foil / Rails (Thin / Medium / Full)
      - Nose Shape (Pointed / Hybrid / Round)
      - Tail Width (Narrow / Medium / Wide)
      - Entry Rocker (Relaxed / Medium / Aggressive)
      - Exit Rocker (Relaxed / Medium / Aggressive)
      - Rocker Style (Staged / Continuous)

    Each scale is a label, a 9px track, a fill, a 19px marker and its scale labels. **Every scale renders in exact proportion to its value (1–100):**
    - The fill is `width: <value>%`.
    - The marker is `left: <value>%` with `transform: translate(-50%, -50%)`, so its centre sits exactly on the value. There is no rounding, rescaling or insetting near the edges. At 1 the marker centre is at 1%; at 100 it is at 100%.
    - Scale labels sit at their true points:
      - the start label at 0% (left-aligned)
      - the middle label, on 3-label scales, centred at 50%
      - the end label at 100% (right-aligned)
    - The 11 numeric fields render their raw value.
    - Skill Level is an enum, mapped Beginner → 1, Intermediate → 50, Advanced → 100.
    - A value outside 1–100 is clamped to that range. This is a guard against bad data only.
    - Each track is `role="meter"` with `aria-valuenow` (the value), `aria-valuemin="1"`, `aria-valuemax="100"` and an accessible name equal to its label.
11. AC11: Buy panel, a bordered 9px-radius box:
    - **Dimensions select:**
      - It has the label "Dimensions" and one option per BoardSize, in ascending length, formatted with `formatBoardSize` (`6'0" · 29.4L`).
      - A size with Stock 0 is a disabled option suffixed " — Sold out".
      - The default is the first in-stock size.
    - **Price** uses `formatPrice`: 24px, weight 700, with a 2px ink underline.
    - **Add to Cart** is the shared `ButtonCTA` in button mode (button-cta AC9), size `md`. It is `type="button"` and does nothing until add-to-cart wires it.
    - **When every size is sold out, or there are no BoardSizes:** the select is disabled (or absent when there are no sizes), and the button reads "Sold out" and is disabled.
    - **Trust list** below a top border, each item with a horizon icon: "Free shipping over $75", "30-day returns", "Ships in 3–5 business days".
12. AC12: Hidden, with no dead UI:
    - the fin setup (diagram, label, blurb and FinSetupNote)
    - the Material and Fin System selects
    - "More options"
    - Wishlist and Share
    - "You might also like"
    - the shaper signature
    - Subtitle
    - the cart drawer

## Technical Approach

**Route: `frontend/src/app/[locale]/products/[productSlug]/page.tsx`**
- `({ params: { productSlug } }) => <ProductDetailPage productSlug={productSlug} />`.
- In Next 14.2 the static `category` segment wins over `[productSlug]`, so the catalog routes are unaffected.

**Data: `frontend/src/lib/strapi/product.ts`** (new; one function per resource)
- `getProductBySlug(slug): Promise<{ kind: 'ok'; product: StrapiProductDetail } | { kind: 'not-found' } | { kind: 'unavailable' }>`.
- Query (`URLSearchParams`):
  ```
  filters[Slug][$eq]=<slug>
  fields[0..4]=Name,Slug,Price,Description,SizeType
  populate[Images][fields][0..2]=url,width,height
  populate[Category][fields][0..1]=Name,Slug
  populate[BoardSizes]=true
  populate[SurfboardSpecs][populate][Video][fields][0..1]=url,mime
  pagination[pageSize]=1
  ```
- It goes through `strapiFetch` (`label: 'product'`, `fallback: null`).
- Results:
  - A `null` or malformed body (`data` is not an array) → `unavailable`.
  - An empty `data` → `not-found`.
  - Otherwise `ok` with `data[0]` cast to the exported raw type `StrapiProductDetail`. The raw type reuses `StrapiBoardSize` from `sizes.ts`.

**View model: `frontend/src/components/product-detail/surfboard-detail.ts`** (pure TS; the mapper lives with the component, per the standard)
- `toSurfboardDetail(raw): SurfboardDetail | null` returns `null` when the product isn't a surfboard or a required field is missing (AC3).
- The view type is:
  ```
  { name, price, category: { name, href }, descriptionMarkdown,
    video?: { src, type },
    images: { src, width, height }[],
    sizes: { label, soldOut }[],
    defaultSizeIndex,
    attributes?: { title, items: { label, value, scale: string[] }[] }[] }
  ```
- Build details:
  - `price` comes from `formatPrice`.
  - Media goes through `strapiMediaUrl`.
  - Sizes are sorted by total length (`boardLengthInches`: `LengthFt × 12 + LengthInches`) and formatted with `formatBoardSize`. `defaultSizeIndex` is the first size with stock, or `-1`.
  - `value` is the raw Strapi number, unrounded. It passes through `clampScale(n) = Math.min(100, Math.max(1, n))` as a guard only.
  - `SKILL_VALUE = { Beginner: 1, Intermediate: 50, Advanced: 100 }`, because SkillLevel is an enum in the schema.
  - Constants: `SCALE_MIN = 1`, `SCALE_MAX = 100`.
- `buildBreadcrumb(detail | null)` → `{ label, href? }[]`.

**Components: `frontend/src/components/product-detail/`** (one component and one return per file)
- **`ProductDetailPage.tsx`** (async server component):
  1. Call `getProductBySlug`.
  2. `unavailable` → unavailable state. `not-found`, or `toSurfboardDetail` returning `null` → `notFound()`.
  3. Render:
     - the breadcrumb (inline map)
     - a 3-column grid made of:
       - the description column: eyebrow, H1, rule, the video conditional, `<SurfboardDescription>`, and the attribute sections inline in maps
       - `<SurfboardGallery>`
       - `<SurfboardBuyPanel>`
  4. Each scale's markup:
     - `<div role="meter" aria-label={label} aria-valuenow={value} aria-valuemin={1} aria-valuemax={100} className="relative h-[9px] rounded ...">`
     - Fill: ``style={{ width: `${value}%` }}``
     - Marker: ``style={{ left: `${value}%` }}`` with `-translate-x-1/2 -translate-y-1/2 top-1/2`
     - Labels: a `relative` row with each label `absolute`:
       - the start label at `left-0`
       - the middle label at `left-1/2 -translate-x-1/2`
       - the end label at `right-0`, with right alignment

     Inline styles are used only for the data-driven percentages.
- **`SurfboardGallery.tsx`** (`'use client'`):
  - Holds `index` and `lightboxOpen` state.
  - The lightbox is inside the same single return, as a conditional.
  - A `useEffect` handles keydown (Esc and arrows), the body `overflow` lock, and focus moving to the close button and back.
  - Images use `next/image` with `width` and `height`.
- **`SurfboardDescription.tsx`** (`'use client'`):
  - Renders `<ReactMarkdown>` inside a `line-clamp-6` wrapper.
  - A ref measures overflow after mount, which decides whether the toggle renders.
  - It holds `expanded` state.
- **`SurfboardBuyPanel.tsx`** (`'use client'`):
  - Holds the selected index in state and renders the `<select>`.
  - The CTA is `<ButtonCTA text=… disabled={!inStock} />`, disabled and reading "Sold out" when nothing is in stock.
  - The trust list is a module-level JSX constant.
- **`product-detail-texts.ts`**:
  - Page: breadcrumb aria-label, Home, unavailable, "From the shaper", "Read more", "Read less".
  - Attributes: "Attributes", the section titles, the 12 attribute labels and their scale labels.
  - Buy panel: "Dimensions", `soldOutOption(label)`, "Add to Cart", "Sold out", the trust items.
  - Gallery and lightbox: "Previous photo", "Next photo", `openPhoto(n, total)`, "Photo viewer", "Close photo viewer", `photoCount(n, total)`.
- **`index.ts`** exports `ProductDetailPage`.

**Shared additions** (additions only; recorded in the owners' Changelogs)
- `shared/components/icons/Icon.tsx`: `close`, `return` and `clock`, using the design's paths. The design's shipping icon is the existing `cart` shape and its read-more chevron is the existing `arrow-right`, so those are reused.
- `frontend/package.json`: add the `react-markdown` dependency.
- Tailwind uses the existing tokens (ink, muted, border, horizon, horizon-deep, horizon-ink, `font-display`, `font-mono`). No config change.

**Tests (TDD, Vitest)**
- `__tests__/lib/strapi/product.test.ts`:
  - Node environment, fetch stubbed as in `products.test.ts`.
  - Checks the query (slug filter, fields, every populate, pageSize 1) and revalidate 60.
  - Checks ok / not-found / unavailable for network failure, 500 and a malformed body.
- `__tests__/components/product-detail/surfboard-detail.test.ts`:
  - The mapper's null cases: a Standard product, no images, no category, NaN price.
  - Size sorting, the sold-out flag, the default index and the all-sold-out case.
  - The 12 attribute items in section order with their labels and scales.
  - Exact values: 34 → 34, 1 → 1, 100 → 100, and a non-integer passes through unrounded. Guard clamping: 0 → 1, 150 → 100.
  - Skill Level: Beginner → 1, Intermediate → 50, Advanced → 100.
  - No SurfboardSpecs → no attributes. Video present and absent.
  - The breadcrumb.
- `__tests__/components/product-detail/ProductDetailPage.test.tsx` (jsdom):
  - Mocks `@/lib/strapi/product` and `next/navigation`.
  - Checks the H1, breadcrumb links, eyebrow, the 3 attribute sections and their 12 meters.
  - Exact rendering for value 34 (fill `width: 34%`, marker `left: 34%`, `aria-valuenow="34"`, `aria-valuemin="1"`, `aria-valuemax="100"`), and the same for 1 and 100.
  - The scale labels at their start, middle and end positions.
  - Checks the video present and absent, `notFound` for not-found and for Standard, and the unavailable state.
  - AC12: no fin setup (no "Thruster" or "fin" label), Material, Fin System, Wishlist, Share, "You might also like", signature or Subtitle renders.
- `SurfboardGallery.test.tsx`: wrap-around prev/next; open, Esc, backdrop click and the close button; arrow keys; the counter; focus return; the single-image case.
- `SurfboardBuyPanel.test.tsx`: options and their order, disabled sold-out options, the default selection, and the all-sold-out button.
- `SurfboardDescription.test.tsx`: markdown renders and raw HTML doesn't; the toggle's `aria-expanded`, with overflow stubbed through the `scrollHeight` / `clientHeight` getters.

## Out of Scope

- **The apparel/standard product page** (SizeType Standard). It is covered by [standard-product-detail-page](../2026-10-07-standard-product-detail-page/spec.md).
- **Add-to-cart behaviour,** the cart drawer and the header count. These belong to the add-to-cart spec.
- **Hidden design features:**
  - the fin setup (diagram, label, blurb, FinSetupNote)
  - Material and Fin System selects
  - Wishlist (customer-account-wishlist) and Share
  - "You might also like"
  - the shaper signature and Subtitle
  - video in the lightbox
- **Per-page `<title>`, SEO metadata and structured data.**
- **Selected size in the URL.**

## Standards Applied

- [frontend/components](../../../../../../standards/frontend/components.md):
  - one component and one return per file
  - a texts file
  - `app/` stays thin
  - Strapi only via `strapiFetch` in `lib/strapi/product.ts`
  - URLs only via `lib/routes.ts`
  - the mapper lives with the component
  - icons only via `Icon`
- [global/git-workflow](../../../../../../standards/global/git-workflow.md): branch and commit conventions.

## Changelog

| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-10-06 | Ori Chai Matan | created | Initial shaping | — |
| 2026-10-06 | Ori Chai Matan | change | Built (T1–T10). Icons: added only `close`, `return` and `clock`; the design's shipping and read-more icons are the existing `cart` and `arrow-right` paths, so `truck` and `chevron-right` weren't added. Select chevron uses the shared `chevron-down` Icon. A scale with a missing value is dropped. Scale labels are a third of the track wide each, anchored at 0 / 50 / 100% | — |
| 2026-10-06 | Ori Chai Matan | change | Size model change (product-content-model): sizes come as `LengthFt` + `LengthInches`; the mapper sorts by `boardLengthInches` and `formatBoardSize` reads the two fields. Tests cover 5'11" < 6'0" < 9'11" < 10'0". Live: tideline, ci-pro, mikey-february-s-shorty and ci-pro-3 show feet + inches labels in order, with sold-out sizes disabled | product-content-model |
| 2026-10-06 | Ori Chai Matan | change | AC11: Add to Cart uses the shared `ButtonCTA` (new button mode, button-cta AC9) instead of a page-local button, so CTA styling lives in one place. Small visual shift from the design's `.btn-add-cart`: 14px text, `.02em` tracking and a 2px transparent border (the ButtonCTA `md` style) instead of 13.5px / `.03em` / no border | button-cta |
| 2026-10-06 | Ori Chai Matan | change | AC7/AC8: gallery frame is fixed (column width × `74vh`, next/image `fill` + `object-contain`) instead of sizing from the photo, so nothing jumps when switching photos; frame and lightbox photo box use the page background token `bg-background` (was hard-coded `#F9F9F9`, the design's page colour). Browser check at 1440/1100/900/390 on ci-pro, big-happy, fever: frame, image centre, prev/next, buy panel, description and the lightbox stage/arrows/counter are identical for every photo. ProductCard (fixed `aspect-[4/5]`) has no jump. Noted: mikey-february-s-shorty's two photos have a baked-in `#F9F9F9` background that shows as a light box on the white page | — |
| 2026-10-06 | Ori Chai Matan | change | `mikey-february-s-shorty`'s two photos (Media Library ids 71 and 72): their baked-in `#F9F9F9` background was made transparent by an edge flood fill (tolerance 10, so the white board interior inside the dark rail is kept), with a 2px colour-to-alpha un-mix on the anti-aliased edge so there is no halo. Same dimensions, PNG with alpha. Replaced in place (same ids, relations and order); originals kept as `seed/catalog/images-local/mikey-february-s-shorty/{1,2}.png.orig`. Verified at 1440 and 390: no grey box, clean nose and rails | — |
| 2026-10-07 | Ori Chai Matan | change | Shared with standard-product-detail-page, with no behaviour change. The 3-column markup moved from `ProductDetailPage` into `product-detail/surfboard/SurfboardDetail.tsx`, and the surfboard files moved into `product-detail/surfboard/`. The breadcrumb (`buildBreadcrumb` + `ProductBreadcrumb`), the lightbox (`PhotoLightbox`, controlled by `SurfboardGallery`), the trust list (`TrustList`) and the unavailable state (`ProductUnavailable`) moved into `product-detail/shared/`. `ProductDetailPage` now switches on `SizeType`, so Standard products render instead of 404ing. Surfboard tests pass unchanged except import paths; "404s for a non-surfboard product" became "404s for a product type no layout renders". The query also requests `Gender` and `StandardSizes` | standard-product-detail-page |
| 2026-10-10 | Ori Chai Matan | change | Add to Cart wired by add-to-cart: `SurfboardBuyPanel` adds the selected board size and opens the cart drawer. The "inert Add to Cart" note is superseded | 2026-10-10-add-to-cart |

# Standard Product Detail Page

> Epic: ../../epic.md
> Status: done
> Branch: feat/storefront-catalog/standard-product-detail-page
> Covers: frontend/src/components/product-detail/standard/**, frontend/src/components/product-detail/shared/**, frontend/src/__tests__/components/product-detail/standard/**, frontend/src/__tests__/components/product-detail/shared/**

## Overview

The detail page at `/products/<productSlug>` for every non-surfboard product (`SizeType = Standard`: clothing, wetsuits, accessories). Today these products 404. This is the second half of the tech plan's `product-detail-page` candidate. surfboard-detail-page delivered the first half, and with this spec a shopper can open any catalog card.

`ProductDetailPage` stays the single entry point. It does one `getProductBySlug` fetch and handles not-found and unavailable. It then switches on `SizeType`: `<SurfboardDetail>` (today's markup, moved with no behaviour change) or `<StandardProductDetail>`.
- Each type keeps its own view model and gallery, because the layouts differ.
- Only the breadcrumb, lightbox, trust list and unavailable state are shared.

Layout (design `westline-site-desing/product-apparel-samurai-boardshort.html`): two columns on desktop.
- **Left:** a 2-up photo grid.
- **Right:** a sticky buy column with eyebrow, H1, price, size grid, Add to Cart, trust list and accordions.

On mobile the gallery is a single-photo swipe (CSS scroll-snap). Clicking a photo opens the lightbox. Add to Cart is inert, as on the surfboard page.

Hidden because Strapi has no field for them, decided in shaping: colour swatch, size chart, tag pill, wishlist, related products.

## Goals & User Stories

- As a shopper, I open a boardshort, wetsuit or accessory from the catalog and see its photos, price, sizes and description, so I can decide whether to buy it.
- As a shopper on desktop, I scroll through all the photos while the buy column stays in view. On mobile, I swipe through them one at a time. I can open any photo full-screen.
- As a shopper, I pick my size from a grid. Sold-out sizes are clearly unavailable, and I'm warned when a size is running low.
- As a content editor, a new standard product in Strapi gets a working page without a deploy.

## Acceptance Criteria

1. AC1: Routing and dispatch.
   - `/products/<slug>` renders a product with `SizeType = Standard` through `StandardProductDetail`. Surfboards render through `SurfboardDetail`.
   - `ProductDetailPage` calls `getProductBySlug` once per request.
   - Every surfboard test still passes with only import paths changed. The one exception is "404s for a non-surfboard product", which this spec replaces.
2. AC2: Data. `getProductBySlug` additionally requests the field `Gender` and `populate[StandardSizes]=true`, still in one request with `revalidate` 60. Strapi itself is unchanged.
3. AC3: 404 (`notFound()`) cases:
   - an unknown slug, as before
   - a `SizeType` that is neither `Surfboard` nor `Standard`
   - a Standard product missing Name, a finite Price, at least one image, or a Category (Name + Slug)
4. AC4: When Strapi is unavailable, the page shows the shared unavailable state with status 200, the Home breadcrumb and "This product couldn't be loaded right now." This is unchanged from the surfboard page.
5. AC5: Breadcrumb (shared component, same styling as the surfboard page).
   - The breadcrumb is `Home / <Category> / <Product>`.
   - When the category is gendered (`GENDERED_CATEGORY_SLUGS`, i.e. clothing) and Gender is Men or Women, a level is added: `Home / Clothing / Men / <Product>`. "Men" links to `genderHref('clothing','men')`.
   - Unisex products, and products without a Gender, get no gender level.
6. AC6: Layout, per design.
   - The container is 1240px max with 24px padding (16px at ≤480px).
   - The grid is `1.6fr 1fr` with a 56px gap.
   - At ≤900px it becomes a single column with a 28px gap, gallery first.
   - Above 900px the buy column is sticky at `top: 88px`. At ≤900px it is static.
7. AC7: Gallery, desktop (>900px).
   - Photos are a 2-column grid with a 3px gap.
   - Each photo is a button labelled "Open photo <n> of <N>". Its frame has a fixed 4:5 aspect ratio and the `bg-background` token, and is never sized from the photo.
   - The photo is a `next/image` with `fill` and `object-contain`, so nothing moves or resizes as images load.
   - The first image loads with `priority`.
   - A product with a single photo spans both columns.
8. AC8: Gallery, mobile (≤900px).
   - It is a horizontal scroller with `scroll-snap-type: x mandatory`.
   - Each photo takes the full width and snaps to centre (`scroll-snap-stop: always`).
   - The scrollbar is hidden. There are no arrows, dots or counter.
   - Tapping a photo opens the lightbox.
9. AC9: Lightbox, shared with the surfboard page. Its behaviour is identical to surfboard AC8:
   - a `dialog` with fixed stage, counter and arrows
   - closes on Esc, the backdrop or the close button
   - arrows wrap
   - scroll lock
   - focus moves to the close button
   - On the standard page, the lightbox opens at the clicked photo, and focus returns to that photo's button on close.
10. AC10: Buy column header.
    - The Category name is a horizon mono eyebrow.
    - The H1 is Poppins 800, uppercase, `clamp(26px,3.4vw,38px)`, line-height 0.95.
    - The price uses `formatPrice`: 24px, weight 700, tabular-nums, no underline.
11. AC11: Size grid.
    - The label row reads "Size — <selected label>".
    - There is one button per StandardSize, in the order S, M, L, XL, One Size, labelled with `formatStandardSize`.
    - The grid has 5 columns (4 at ≤420px) and an 8px gap. Each box is a 9px-radius bordered box in mono 13px/600.
    - The selected size is ink-filled with white text and `aria-pressed="true"`.
    - The default selection is the first in-stock size.
12. AC12: Sold out and low stock.
    - **Sold out (Stock 0):** a disabled button with muted text and a line-through, accessible name "<size> — Sold out".
    - **Low stock (Stock 1–3, constant `LOW_STOCK_MAX = 3`):** a 15px `danger`-token "!" flag on the box's top-right corner, accessible name "<size> — Low stock".
    - When the selected size is low, a "Low stock" note (mono 11.5px, uppercase, `text-danger`) shows under the grid.
    - When every size is sold out, or there are no sizes: no selectable size, and the CTA reads "Sold out" and is disabled.
13. AC13: Add to Cart is the shared `ButtonCTA` in button mode, size `md`, and inert (no handler) until add-to-cart wires it.
14. AC14: The trust list is shared and its text is identical to the surfboard page's: "Free shipping over $75", "30-day returns", "Ships in 3–5 business days".
15. AC15: Accordions sit below the buy panel.
    - **Description:** open by default. The markdown is rendered with `react-markdown`, with no raw HTML.
    - **Shipping & Returns:** static texts, "Free shipping on orders over $75." and "30-day returns."
    - Only one accordion is open at a time, and clicking the open one closes it.
    - Each header is a button with `aria-expanded` and `aria-controls`. A 16px chevron rotates 180° when open.
    - Styling follows the design: 15px/700 headers with 20px vertical padding, a border-bottom per group, and a muted 14.5px/1.7 body.
16. AC16: Hidden, with no dead UI: colour swatch, Size Chart, tag pill / Subtitle, Wishlist, "You might also like", quantity, the "Details & Features" accordion, the "See details" links, and the cart drawer.

## Technical Approach

**Folder layout:** `frontend/src/components/product-detail/`
- `ProductDetailPage.tsx`, `index.ts`, `product-detail-texts.ts`
- `shared/`
  - `breadcrumb.ts`: `buildBreadcrumb`, moved from `surfboard-detail.ts`
  - `ProductBreadcrumb.tsx`, `PhotoLightbox.tsx`, `TrustList.tsx`, `ProductUnavailable.tsx`
- `surfboard/`
  - `SurfboardDetail.tsx`: the current 3-column grid
  - `surfboard-detail.ts`, `SurfboardGallery.tsx`, `SurfboardBuyPanel.tsx`, `SurfboardDescription.tsx`
- `standard/`
  - `standard-detail.ts`, `StandardProductDetail.tsx`, `StandardGallery.tsx`, `StandardBuyPanel.tsx`, `StandardAccordions.tsx`

One texts file stays at the folder root and gains a `standard` block for: size label, sold out / low stock labels, accordion titles and the shipping copy.

**Data: `lib/strapi/product.ts`**
- Add `fields[5]=Gender` and `populate[StandardSizes]=true`.
- `StrapiProductDetail` gains `Gender?: StrapiGender | null` (from `lib/routes.ts`) and `StandardSizes?: StrapiStandardSize[] | null` (from `sizes.ts`).

**`ProductDetailPage.tsx`** (async server component, one return):
1. Fetch the product. `not-found` → `notFound()`. `unavailable` → `<ProductUnavailable/>`.
2. Switch on `SizeType`:
   - `'Surfboard'`: `toSurfboardDetail`
   - `'Standard'`: `toStandardDetail`
   - Anything else, or a `null` mapping, → `notFound()`.
3. Render `<ProductBreadcrumb items={buildBreadcrumb(...)}/>` followed by the type's component.

**`shared/`**
- **`buildBreadcrumb(input | null)`:** the input is `{ name, category: { name, href }, gender?: { label, href } }`. With `null` it returns `[Home]`, as today.
- **`PhotoLightbox`** (`'use client'`):
  - Controlled. Props: `{ images, index, onIndexChange, onClose }`.
  - It holds the keydown / scroll-lock / focus-close `useEffect` and the markup moved verbatim from `SurfboardGallery`.
  - The parent restores focus in `onClose`.
- **`TrustList`:** the JSX constant from `SurfboardBuyPanel`.
- **`ProductUnavailable`:** the breadcrumb plus the message.

**`surfboard/`**
- The files move as they are.
- `SurfboardGallery` renders `<PhotoLightbox>` in place of its inline lightbox. `SurfboardBuyPanel` renders `<TrustList/>`.
- `SurfboardDetail({ detail })` is the grid markup cut from `ProductDetailPage`, including the attribute meters.

**`standard/standard-detail.ts`** (pure TS)
- `toStandardDetail(raw): StandardDetail | null`. It returns null for a non-Standard SizeType or a missing required field.
- View type:
  ```
  { name, price, category: { name, href }, gender?: { label, href },
    descriptionMarkdown, images: { src, width, height }[],
    sizes: { label, soldOut, lowStock }[], defaultSizeIndex }
  ```
- Constants: `SIZE_ORDER = ['S','M','L','XL','OneSize']` and `LOW_STOCK_MAX = 3`.
- Media goes through `strapiMediaUrl`. Prices use `formatPrice`. Gender links use `genderHref` and require `GENDERED_CATEGORY_SLUGS` to include the slug.

**`standard/` components** (one component and one return per file)
- **`StandardProductDetail`** (server): the 2-column grid with `<StandardGallery>`, then the sticky buy column. The buy column holds the eyebrow, H1, price, `<StandardBuyPanel>`, `<TrustList/>` and `<StandardAccordions>`.
- **`StandardGallery`** (`'use client'`):
  - State: `lightboxIndex: number | null`, plus an array of item refs used to return focus.
  - Desktop: `grid grid-cols-2 gap-[3px]`.
  - ≤900: `max-[900px]:flex overflow-x-auto snap-x snap-mandatory`, items `flex-[0_0_100%] snap-center snap-always`, scrollbar hidden through arbitrary variants.
- **`StandardBuyPanel`** (`'use client'`):
  - Holds the selected index, the size grid, the low-stock note and `ButtonCTA`.
  - Tailwind uses the existing tokens plus a new `danger` colour token (`#C0392B`, the design's low-stock red) added to `frontend/tailwind.config.ts`, used as `bg-danger` / `text-danger`. Arbitrary values only for the percent / px sizes.
  - `ButtonCTA` button mode already supports `disabled` (`disabled:bg-muted`, button-cta AC9), so it is used unchanged.
- **`StandardAccordions`** (`'use client'`):
  - Holds `openId` state, defaulting to `'description'`.
  - The two groups are rendered inline from a constant list.
  - The chevron is the existing `chevron-down` Icon.

**Tests (TDD, Vitest).** New tests live under `__tests__/components/product-detail/{shared,standard}/`. Surfboard tests stay where they are; only their import paths change.

## Out of Scope

- Changes to the Strapi schema, including Color, Tag and size-chart fields.
- **Hidden design items:**
  - colour swatch
  - size chart
  - tag pill / Subtitle
  - wishlist (customer-account-wishlist)
  - "You might also like"
  - quantity
  - the Details & Features accordion (that content stays inside Description)
  - "See details" links
- Add-to-cart behaviour and the cart drawer (the add-to-cart spec).
- Merging the two galleries or the two buy panels.
- A full-width Add to Cart: ButtonCTA keeps its natural width, as on the surfboard page.
- SEO metadata.
- The selected size in the URL.
- Dark-mode tuning of `bg-background`, which goes near-black under `prefers-color-scheme: dark`, as on the surfboard page.

## Standards Applied

- [frontend/components](../../../../../../standards/frontend/components.md):
  - one component and one return per file
  - the texts file
  - `app/` stays thin
  - Strapi only via `strapiFetch`
  - URLs only via `lib/routes.ts`
  - the mapper lives next to its component
  - icons only via `Icon`
- [global/git-workflow](../../../../../../standards/global/git-workflow.md)

## Changelog

| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-10-07 | Ori Chai Matan | created | Initial shaping | — |
| 2026-10-07 | Ori Chai Matan | change | Review: added a `danger` colour token (`#C0392B`) to `frontend/tailwind.config.ts` for the low-stock flag and note instead of a hard-coded hex; ButtonCTA confirmed to support `disabled` (no change); surfboard regression checks use ci-pro and mikey-february-s-shorty (tideline was deleted) | — |
| 2026-10-07 | Ori Chai Matan | change | Built (T1–T13). Shared extras beyond the plan: `shared/layout.ts` holds the `CONTAINER` class both layouts and the breadcrumb use. `PhotoLightbox` is controlled (`index` / `onIndexChange` / `onClose`); the parent restores focus. The size group is `role="group"` named by its "Size — <label>" label | — |
| 2026-10-07 | Ori Chai Matan | change | QA (T14): PASS WITH NOTES, see qa-report.md. Status → done | — |
| 2026-10-10 | Ori Chai Matan | change | AC13 superseded by add-to-cart: Add to Cart adds the selected size and opens the cart drawer. Trust/shipping copy now reads `FREE_SHIPPING_THRESHOLD` | 2026-10-10-add-to-cart |

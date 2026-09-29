# Product Card

> Epic: ../../epic.md
> Status: in-progress
> Branch: feat/storefront-catalog/product-card
> Covers: frontend/src/shared/components/product-card/**, frontend/src/lib/strapi/media.ts, frontend/src/__tests__/shared/components/product-card/**, frontend/src/__tests__/lib/strapi/media.test.ts

## Overview

This spec delivers the single product card that the catalog listing, the Home "Featured gear" section and the wishlist all render: "one product card" per the brief. It adds:
- the card component
- a `ProductCard` view type
- a mapper from the Strapi Product (see product-content-model) to that type

The card was extracted from catalog-listing-page so it can be reviewed and reused on its own; it isn't in the tech plan's Candidate Specs list. The design source is the canonical `.prod-card` in `westline-site-desing/index.html` (Featured gear). The simpler card in `catalog.html` is superseded.

No page renders the card yet. Tests verify it, and catalog-listing-page will be the first page to use it.

## Goals & User Stories

- As a shopper, I see each product as a consistent card:
  - a 4:5 photo I can flip through when there are several
  - the product name and price
  - the whole card links to the product page
- As a signed-in shopper (once customer-account-wishlist wires it), I can favorite a product from its card. The heart shows whether it's saved.
- As a developer on the listing, Home and wishlist specs, I render one `<ProductCard>` from a `ProductCard` value built by one mapper. I don't re-derive image URLs, image fit or price formatting.

## Acceptance Criteria

1. AC1: `toProductCard(product)` in `frontend/src/shared/components/product-card/product-card.ts` maps a Strapi REST Product (`Name`, `Slug`, `Price`, `Images[]`, `Category.Slug`) to `ProductCard`:
   - `{ slug, name, price, href: '/products/<slug>', images: { src, width, height }[], imageFit: 'contain' | 'cover' }`
   - It keeps at most the first 4 images, in order.
   - `Price` is coerced with `Number()`, because Strapi decimals can arrive as strings with Postgres (`"829.00"` → 829 → `$829`).
   - It returns `null` when `Name` or `Slug` is missing, when `Price` doesn't coerce to a finite number, or when `Images` is empty or missing. Images are required in Strapi, so the card has no empty-image state.
2. AC2: Relative Strapi upload URLs (`/uploads/…`) become absolute against `STRAPI_URL` through `strapiMediaUrl` (`frontend/src/lib/strapi/media.ts`). Absolute URLs pass through unchanged.
3. AC3: `imageFit` is `contain` when the product's Category slug is `surfboards`, otherwise `cover`, including when the Category is missing. It's derived from the Category, not an editor field.
4. AC4: The card shows:
   - a 4:5 image area on the light surface background, with the photos fitted per `imageFit`
   - the name as an `<h3>`
   - the price, formatted as USD with no cents for whole amounts (`$829`, `$79`) and two decimals otherwise (`$49.50`)
   - a 1px border and 9px radius, as in the design
5. AC5: With 2–4 images, the photos stack. Only the active one is visible, and it fades with a short opacity transition that is removed under reduced motion.
   - Photo dots ("Show photo N") appear bottom-center over the image. Clicking a dot shows that photo and sets `aria-current="true"` on that dot only.
   - With 1 image there are no dots. A card always has at least 1 image (AC1).
   - The card never shows more than 4 photos or dots, even if given more.
6. AC6: The first photo has `alt` = the product name. Extra photos have `alt=""`. Only the first photo can load with priority (via a `priority` prop), and the rest lazy-load.
7. AC7: The whole card is a link to `/products/<slug>`, made with a full-card overlay `next/link` whose visually hidden text is "View <name>". The dots and heart sit above the overlay and don't trigger navigation. The link will 404 until product-detail-page exists, which is accepted.
8. AC8: The wishlist heart (top-right) renders **only** when both `isFavorite` and `onToggleFavorite` are passed, so there's never a dead button.
   - It has `aria-pressed={isFavorite}` and the label "Add to wishlist" or "Remove from wishlist".
   - The icon is filled when pressed.
   - Clicking it calls `onToggleFavorite(slug)`. The card holds no favorite state of its own; it's fully controlled by its parent.
9. AC9: Only the photo stack, dots and heart are a client component. The rest of the card is plain React without server-only APIs, so it works from both Server Components and client trees. `onToggleFavorite` is a function, so the heart can only be wired from a client parent (the wishlist).

## Technical Approach

**Data (`frontend/src/shared/components/product-card/product-card.ts`)**
- `export interface ProductCard { slug; name; price; href; images: { src; width; height }[]; imageFit: 'contain' | 'cover' }`
- `export interface StrapiProduct`: the minimal REST shape (`Name`, `Slug`, `Price`, `Images?: { url; width; height }[]`, `Category?: { Slug } | null`).
- `export function toProductCard(product: StrapiProduct): ProductCard | null` implements AC1–AC3:
  - `Price: number | string` is converted with `const price = Number(product.Price)`, and the function returns `null` unless `Number.isFinite(price)`.
  - It returns `null` when `Images` is empty or missing.
  - `imageFit` is `product.Category?.Slug === 'surfboards' ? 'contain' : 'cover'`.
  - Observed live shape (2026-09-28, local Postgres): `GET /api/products` returned `Price` as a JSON number (`79`, `829`). The string branch is defensive, for other drivers or configs.
- `export function formatPrice(price: number): string`: `Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD', minimumFractionDigits: Number.isInteger(price) ? 0 : 2 })`.
- Callers fetch with `populate[Images][fields][0..2]=url,width,height&populate[Category][fields][0]=Slug`. The listing spec owns fetching; this spec only documents the populate it needs.

**URL helper (`frontend/src/lib/strapi/media.ts`)**
- `strapiMediaUrl(url)`: a relative `/…` URL gets `STRAPI_URL` (default `http://localhost:1337`, same as `homepage.ts`) prefixed; anything else is returned as is.
- This file is server-and-client safe: `STRAPI_URL` is only read on the server during mapping, and the mapper runs where the data is fetched.
- `frontend/src/lib/strapi/homepage.ts` (owned by home-hero) imports `strapiMediaUrl` and its local `absoluteUrl` is removed. Its behavior doesn't change, and `homepage.test.ts` plus the hero tests must stay green.

**Component (`frontend/src/shared/components/product-card/`)**
- `ProductCard.tsx` is the shell, with no `'use client'` and no server-only imports:
  - `<article>` with `relative overflow-hidden rounded-[9px] border border-border bg-white`
  - an overlay `<Link href={card.href} className="absolute inset-0 z-[1]"><span className="sr-only">View {name}</span></Link>`
  - `<ProductCardMedia …/>`
  - a body (`p-4`) with an `<h3 className="mb-1.5 text-base font-bold">` and a price `<span className="text-base">`
  - Props: `{ product: ProductCard; priority?: boolean; isFavorite?: boolean; onToggleFavorite?: (slug: string) => void }`
- `ProductCardMedia.tsx` (`'use client'`) holds the active-index state:
  - Up to 4 `next/image` elements (`fill`, `sizes="(max-width: 520px) 100vw, (max-width: 1024px) 50vw, 25vw"`) with `object-contain` or `object-cover`. Inactive ones get `opacity-0`, with `transition-opacity duration-[250ms] motion-reduce:transition-none`.
  - Dots only when there are 2 or more images (`aria-label="Show photo N"`, `aria-current`). They're styled like the design: a pill container, bg white/85, with the active dot a wide 22px pill in ink.
  - The heart button only when both favorite props are present.
  - The dots and heart are `z-[2]` so they sit above the overlay link.
- `index.ts` re-exports `ProductCard`.
- The heart SVG path is copied from the design. Colors use the existing Tailwind tokens (`ink`, `horizon`, `border`) and `#F9F9F9` for the media background.

**Tests (TDD, `frontend/src/__tests__/…`)**
- `lib/strapi/media.test.ts` and `shared/components/product-card/product-card.test.ts` (node environment): AC1–AC3 and `formatPrice`.
- `shared/components/product-card/ProductCard.test.tsx` (jsdom + RTL): AC4–AC8, using real `next/image` and `next/link` (both work in the existing Vitest setup, as the hero tests show).

## Out of Scope

- The catalog listing page, the grid, fetching product lists, filters and sort (catalog-listing-page, catalog-search-filter-sort).
- The Home "Featured gear" section and its slider (arrows, track dots), which belong to a Home section spec.
- Favorites state, persistence and auth (customer-account-wishlist). The card only renders what it's given.
- The PDP at `/products/[slug]` (product-detail-page).
- Add to cart from the card, and the design's unused `.tag` / `.spec` / `.btn-add` styles.
- Switching links to next-intl (a separate i18n spec if ever needed).

## Standards Applied

- [global/git-workflow](../../../../../../standards/global/git-workflow.md): branch `feat/storefront-catalog/product-card`, commit and PR conventions.

## Changelog

| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-09-28 | Ori Chai Matan | created | Initial shaping | — |
| 2026-09-29 | Ori Chai Matan | change | Mapper + view type moved from `frontend/src/features/catalog/product-card.ts` to `frontend/src/shared/components/product-card/product-card.ts` (test: `frontend/src/__tests__/shared/components/product-card/product-card.test.ts`); `src/features/` removed — everything the card needs (component, view type, Strapi → card mapper) lives in one folder and is imported from one place | — |

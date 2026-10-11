# Catalog Listing Page

> Epic: ../../epic.md
> Status: in-progress
> Branch: feat/storefront-catalog/catalog-listing-page
> Covers: /products, /products/category/*, frontend/src/app/[locale]/products/page.tsx, frontend/src/app/[locale]/products/category/**, frontend/src/components/catalog/**, frontend/src/lib/strapi/products.ts, frontend/src/__tests__/components/catalog/**, frontend/src/__tests__/lib/strapi/products.test.ts

## Overview

A server-rendered catalog listing at `/products` (all products) and `/products/category/<categorySlug>`. It serves the URLs that site-nav already links to:
- `?sub=<subcategorySlug>`
- `?gender=men|women`, on Clothing only
- `?page=<n>`

The page shows:
- a product grid built from the existing ProductCard
- a breadcrumb, the category heading and a result count
- a category-tree sidebar built from the same Strapi data as the nav
- numbered pagination

It is the tech plan's `catalog-listing-page` candidate and the first step of the brief's success path (land on the catalog → …). Search, filters and sort are catalog-search-filter-sort.

The design source is `westline-site-desing/catalog.html` (`.cat-header`, `.catalog-grid`, `.catalog-sidebar`, `.cat-grid-products`, `.pagination`). It is matched as-is except for the hidden items (AC12) and the gaps the design leaves open, which were decided in shaping: Prev, the empty state, the unavailable state and mobile sidebar visibility.

## Goals & User Stories

- As a shopper, I open a category from the header and see its products in a grid, 16 per page, and page through them.
- As a shopper, I narrow to a subcategory, or for Clothing to Men or Women, from the sidebar or the header, and always see where I am (breadcrumb, heading, active sidebar item).
- As a shopper, when a subcategory has nothing yet, I'm told so and offered the whole category.
- As a content editor, a new category or subcategory in Strapi shows up in the sidebar and gets a working URL without a deploy.

## Acceptance Criteria

1. AC1: Routes. `/products` lists every product. `/products/category/<categorySlug>` lists one category. Both are server-rendered. Every URL on the page comes from `lib/routes.ts` (`PRODUCTS_HREF`, `catalogHref`, and the existing builders).
2. AC2: The data comes from Strapi through `strapiFetch`, with server-side filters:
   - Category slug.
   - Subcategory slug (`?sub`).
   - Gender. `?gender=men` → Gender in [Men, Unisex], `women` → [Women, Unisex], via `GENDER_FILTER`.
   - Order is `sort=id:asc`, so pages never repeat or skip a product.
   - `PAGE_SIZE = 16`, defined once in `lib/strapi/products.ts`.
3. AC3: Gender applies only to Clothing (`GENDERED_CATEGORY_SLUGS`). On any other category, and on `/products`, `?gender` is ignored (no 404). An unrecognized `?gender` value on Clothing is also ignored. Links for non-Clothing categories never include `gender`.
4. AC4: Invalid params → 404 (`notFound()`):
   - An unknown category.
   - A `?sub` that isn't one of this category's subcategories.
   - A `?page` that isn't an integer ≥ 1 (`0`, `-1`, `abc`, `1.5`).
   - A `?page` above the last page. Page 1 always exists, so an empty list shows the empty state on page 1 and 404s on `?page=2`.

   A missing `?page` means page 1. `?sub` on `/products` is ignored. With an array param (`?sub=a&sub=b`), the first value is used.
5. AC5: Grid: 4 columns above 1080px, 3 up to 1080px, 2 at 760px and below (gap 20px, then 14px), as in the design. Cards are the shared `ProductCard`, mapped with `toProductCard`, and **no heart renders** (no favorite props are passed). The first 4 cards (the first desktop row) load with `priority`.
6. AC6: Chrome, per design:
   - The breadcrumb is JetBrains Mono, 12px, uppercase, muted, with a `/` separator and the current item in ink. It reads:
     - `/products`: Home / All Products
     - a category: Home / <Category>
     - a subcategory: Home / <Category> / <Subcategory Name>
     - Clothing with a gender: Home / Clothing / Men, optionally / <Subcategory>
     Every item except the last is a link.
   - The H1 is Poppins 800, uppercase, `clamp(30px,4.5vw,52px)`. It is the category name, or "All Products" on `/products`.
   - The toolbar shows "<N> results" ("1 result"), where N is the total across all pages. It is hidden when N is 0 (the empty state, AC9).
7. AC7: Pagination:
   - Page numbers are centered 36×36 links, and the current page is ink-filled with `aria-current="page"`.
   - A Prev chevron (not rendered on page 1) and a Next chevron (not rendered on the last page) sit at the ends.
   - Links keep `?sub` and a valid Clothing `?gender`. Page 1 links carry no `?page`.
   - No pagination renders when there is a single page. With today's 17 products, `/products` has 2 pages.
8. AC8: Sidebar (above 900px only; hidden at ≤900px, where the header's mobile menu covers browsing):
   - It has a "Categories" title and one accordion group per Strapi category, in Strapi order, built from `getNavigation()` + `buildNavMenu()` (the same data as the nav).
   - Each group's parent is a toggle button with `aria-expanded` and a chevron. Its children are "All <Category>", then its subcategories (`NavLabel ?? Name`).
   - Clothing instead lists "Shop All Clothing", then nested **Men** and **Women** toggles. Each lists "All Men" / "All Women" (the `?gender` URL), then the subcategories relevant to that gender (as in site-nav AC3).
   - The active link (the one whose href matches the current category / sub / gender) is horizon blue, weight 600, with a 2px left border, and has `aria-current="page"`. Its group, and its gender subgroup if any, start open. The other groups start closed.
   - On `/products`, nothing is active and all groups start closed.
9. AC9: Empty result: when a valid category or subcategory has no products, the grid area shows "No products here yet" and a link "All <Category>" to the category URL. On `/products`, only the message shows. There is no result count and no pagination.
10. AC10: Strapi unavailable (the category list comes back empty, or the product request fails): the page still renders with status 200. It shows the breadcrumb (Home), no sidebar and no count, and the message "Products couldn't be loaded right now." It never 404s because of an outage.
11. AC11: Layout per design: the container is 1240px max with 24px padding (16px at ≤480px). The sidebar column is 248px with a 40px gap. The header section has 28/20px padding and a bottom border, and the catalog section has 36/80px padding.
12. AC12: Hidden, with no dead UI: the filter accordions, the sort select, the mobile Filter drawer and its button, the "Take the 2-Minute Finder" callout, and the card heart.

## Technical Approach

**Routes: `frontend/src/app/[locale]/products/`** (explicit folders beat `[[...segments]]`)
- `page.tsx` → `<CatalogPage searchParams={searchParams} />`.
- `category/[categorySlug]/page.tsx` → `<CatalogPage categorySlug={params.categorySlug} searchParams={searchParams} />`.
- Both are one-liners. Next 14.2 passes `params` and `searchParams` as plain objects, and reading `searchParams` makes the page dynamic.

**URLs: `frontend/src/lib/routes.ts`** (site-nav's file; additions only, recorded in its Changelog)
- `PRODUCTS_HREF = '/products'`.
- `catalogHref({ categorySlug?, subcategorySlug?, gender?, page? })` builds the path, then query params in the order `sub`, `gender`, `page`:
  - `gender` is dropped unless the category is in `GENDERED_CATEGORY_SLUGS`.
  - `page` is omitted when it is 1.
  - The output equals the existing `categoryHref` / `subcategoryHref` / `genderHref` / `subcategoryGenderHref` strings for the same inputs.

**Data: `frontend/src/lib/strapi/products.ts`** (new)
- `PAGE_SIZE = 16`.
- `getProductList({ categorySlug?, subcategorySlug?, genders?, page }): Promise<ProductList | null>`, where `ProductList = { products: ProductCard[]; total: number; pageCount: number }`.
- The query:
  `products?fields[0]=Name&fields[1]=Slug&fields[2]=Price&populate[Images][fields][0..2]=url,width,height&populate[Category][fields][0]=Slug&filters[Category][Slug][$eq]=…&filters[Subcategories][Slug][$eq]=…&filters[Gender][$in][i]=…&sort[0]=id:asc&pagination[page]=n&pagination[pageSize]=16`
  Filters are included only when set. It goes through `strapiFetch` (`revalidate` 60, `fallback: null`).
- It validates `data[]` + `meta.pagination`, maps with `toProductCard` (from `@/shared/components/product-card/product-card`) and drops nulls. It returns `null` on a failure or a malformed body.

**Page model: `frontend/src/components/catalog/catalog.ts`** (pure TS)
- `resolveCatalogParams(categories, categorySlug, searchParams)` → `{ kind: 'not-found' } | { kind: 'ok', category?, subcategory?, gender?, page }` (AC3, AC4).
- `buildBreadcrumb(resolved)` (AC6).
- `buildSidebar(navItems, location)` maps site-nav's `NavItem[]` to groups / subgroups / links with `active` and `open` flags (AC8).
- `buildPagination(resolved, pageCount)` → `{ prevHref?, nextHref?, pages: { number, href, current }[] }` (AC7).

**Components: `frontend/src/components/catalog/`** (one component, one return per file)
- `CatalogPage.tsx` (async server component) follows these steps:
  1. Call `getNavigation()`. Next dedupes it with SiteNav's identical fetch.
  2. If there are no categories → the unavailable state.
  3. Call `resolveCatalogParams` → `notFound()` if the params are invalid.
  4. Call `getProductList`.
  5. `null` → the unavailable state. Page > `max(1, pageCount)` → `notFound()`.
  6. Render the header (breadcrumb inline in a map, H1), then the grid shell: `<CatalogSidebar>` and the main column (toolbar count, `ProductCard` map with `priority={i < 4}` or the empty / unavailable variant, `<CatalogPagination>`).
- `CatalogSidebar.tsx` (`'use client'`) keeps a `Set` of open group/subgroup keys, seeded from the model's `open` flags. Groups and subgroups are inline in maps. It is hidden at ≤900px with `max-[900px]:hidden`.
- `CatalogPagination.tsx` (server) renders `next/link` numbers plus Prev/Next as `<Icon name="arrow-left|arrow-right">`. Those paths are exactly the design's chevrons.
- `catalog-texts.ts`: "Home", "All Products", "Categories", `results(n)`, `allCategory(name)`, `shopAll(name)`, `allGender(label)`, "No products here yet", "Products couldn't be loaded right now.", the Breadcrumb / Pagination aria-labels, "Previous page" / "Next page", `pageLabel(n)`. The sidebar toggles are named by their visible label, so they need no text.
- `index.ts` exports `CatalogPage`.
- `shared/components/icons/Icon.tsx`: add `chevron-down` (`M6 9l6 6 6-6`) for the sidebar toggles.
- Tailwind uses the existing tokens (ink, muted, border, horizon, `font-display`, `font-mono`). No config change.

**Tests (TDD, Vitest):**
- `__tests__/lib/strapi/products.test.ts` (node environment, fetch stubbed as in `navigation.test.ts`).
- `__tests__/lib/routes.test.ts`: extended for `catalogHref`.
- `__tests__/components/catalog/catalog.test.ts`: params, breadcrumb, sidebar, pagination, including a mocked 100-product / 7-page case.
- `__tests__/components/catalog/CatalogPage.test.tsx` (jsdom): mock `@/lib/strapi/navigation`, `@/lib/strapi/products` and `next/navigation` (`notFound` throws), then `render(await CatalogPage(...))`.
- `__tests__/components/catalog/CatalogSidebar.test.tsx`.

## Out of Scope

- Search, filters, sort, the mobile Filter drawer and search no-results (catalog-search-filter-sort).
- The Finder callout (surfboard-finder epic) and favorites / wishlist hearts (customer-account-wishlist).
- A mobile category tree (the site-nav hamburger covers it), an ellipsis in pagination, and per-page `<title>` metadata.
- The PDP at `/products/[slug]` (product-detail-page) and add-to-cart.
- Custom or editor-controlled ordering. Order is `id:asc`.

## Standards Applied

- [frontend/components](../../../../../../standards/frontend/components.md): one component and one return per file, texts files, `app/` thin, Strapi calls only via `lib/strapi/client.ts`, URLs only via `lib/routes.ts`, the mapper stays with product-card.
- [global/git-workflow](../../../../../../standards/global/git-workflow.md): branch and commit conventions (no commits in this build).

## Changelog

| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-09-29 | Ori Chai Matan | created | Initial shaping | — |
| 2026-09-29 | Ori Chai Matan | change | Built (T1–T10): AC4 now states that `?page` above `max(1, pageCount)` 404s, including `?page=2` on an empty list; `buildSidebar` takes the resolved location (it derives the current href); dropped the unused `toggleGroup` text | — |
| 2026-09-29 | Ori Chai Matan | change | AC6/AC9: the "N results" count is hidden when the result is 0 (empty state) | — |

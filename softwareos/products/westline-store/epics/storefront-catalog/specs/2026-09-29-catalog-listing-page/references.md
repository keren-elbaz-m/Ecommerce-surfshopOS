# References for Catalog Listing Page

## Related Specs

- **[Related]** [2026-09-29-site-nav](../2026-09-29-site-nav/spec.md): this page serves site-nav's URL scheme (`categoryHref`, `subcategoryHref`, `genderHref`, `subcategoryGenderHref`). Its sidebar is built from site-nav's data and model (`getNavigation()` + `buildNavMenu()`). This spec adds `PRODUCTS_HREF` and `catalogHref` to `lib/routes.ts` and `chevron-down` to `shared/components/icons/Icon.tsx`, both owned by site-nav, and records them in site-nav's Changelog. It matched in Step 4 on a body mention of `/products/category/…`. That was dismissed as a different concern: site-nav lists "the pages behind the links" as Out of Scope.
- **[Related]** [2026-09-28-product-card](../2026-09-28-product-card/spec.md): `ProductCard` and `toProductCard` are reused unchanged. The heart doesn't render because no favorite props are passed. It matched on `/products/<slug>` (the PDP link) and was dismissed as a different concern.
- **[Related]** [2026-09-28-product-content-model](../2026-09-28-product-content-model/spec.md) is the data source: Product → Category (many-to-one), Subcategories (many-to-many), and Gender. Its lifecycle rule allows Gender only on Clothing. It matched on `GET /api/products` and was dismissed as a different concern.
- **[Related]** catalog-search-filter-sort (not shaped yet) will add the filter accordions, the sort select, the mobile Filter drawer and button, search, and search no-results to this page.

## Upstream Meeting

_None._

## Similar Implementations

### Filtered, paged product query

- **Location:** `frontend/src/lib/strapi/navigation.ts` (`getSubcategoryGenders`)
- **Key patterns:** a `URLSearchParams` query with `filters[Category][Slug][$in]`, `pagination[page]`/`pageSize`, and reading `meta.pagination.pageCount`. Every request goes through `strapiFetch`.

### Strapi resource function

- **Location:** `frontend/src/lib/strapi/homepage.ts`
- **Key patterns:** a query constant, `strapiFetch` with `fallback: null`, and shape validation followed by a normalizer that filters out nulls.

### Async server component + tests

- **Location:** `frontend/src/components/site-nav/SiteNav.tsx`, `frontend/src/__tests__/components/site-nav/SiteNav.test.tsx`, `frontend/src/__tests__/components/home/hero/Hero.test.tsx`
- **Key patterns:** `vi.mock` the data module, then `render(await Component())`. Fetch tests stub `fetch` in the node environment.

### Route layout

- **Location:** `westline-workplace/Claude outputs/folder-structure.md` (adopted 2026-09-27)
- **Key patterns:** `[locale]/products/page.tsx` and `[locale]/products/category/[categorySlug]/page.tsx`. Explicit folders take priority over `[[...segments]]`, and `page.tsx` stays thin. Its `features/` layout is superseded by `standards/frontend/components.md`.

## Visuals

- `westline-workplace/westline-site-desing/catalog.html` is referenced by path, not copied. `visuals/catalog-notes.md` has the extracted CSS, markup and behavior.

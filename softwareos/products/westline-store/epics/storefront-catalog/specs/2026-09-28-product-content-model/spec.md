# Product Content Model

> Epic: ../../epic.md
> Status: in-progress
> Branch: feat/storefront-catalog/product-content-model
> Covers: cms/src/api/category/**, cms/src/api/subcategory/**, cms/src/api/product/**, cms/src/components/product/**, cms/src/bootstrap/catalog.ts, cms/src/bootstrap/public-permissions.ts, cms/seed/catalog/**, cms/scripts/seed-catalog.js, cms/scripts/verify-catalog.mjs

## Overview

Adds the Strapi content model for WESTLINE's product catalog: `Product`, `Category` and `Subcategory` collection types, seeded with 4 categories / 16 subcategories / 17 products, exposed read-only over public REST. This is the data layer the catalog-listing-page, catalog-search-filter-sort and product-detail-page specs will build on — no frontend consumes it yet.

Categorization is a **flat two-level model** — every product has exactly one main `Category` and zero or more `Subcategory` entries, each of which belongs to exactly one `Category`. A self-referencing `Category` tree (`Parent`/`Children`, walked with a multi-depth `$or` ancestor filter to answer "all products under Surfboards") was the first idea but wasn't built: two collection types with a single `manyToOne`/`manyToMany` hop answer both "main category" and "subcategory" filters directly, with no recursive query and no depth limit to pick. `Subcategory` already carries which `Category` it belongs to, so a product's Subcategories are cross-checked against its Category by a lifecycle hook rather than trusted to whoever fills in the admin form.

## Goals & User Stories

- As a shopper (in later specs), I can browse by main category (e.g. "Surfboards") or drill into a subcategory (e.g. "Soft-top / Beginner") and see the right products either way.
- As a content editor, I add a product, pick its one Category and any number of matching Subcategories, and Strapi stops me if a Subcategory doesn't belong to that Category.
- As a developer building the catalog pages, I can fetch products/categories/subcategories over public REST with ordinary Strapi filters — no bespoke endpoint, no recursive tree query.

## Acceptance Criteria

1. AC1: `Category` (`api::category.category`) is a flat collection type: `Name` (string, required), `Slug` (uid from `Name`, required). No self-relation (no `Parent`/`Children`).
2. AC2: `Subcategory` (`api::subcategory.subcategory`) is a collection type: `Name` (string, required), `Slug` (uid from `Name`, required), `NavLabel` (string, optional — the short menu label site-nav shows, falls back to `Name`; set only on the Wetsuits subcategories, "Men" / "Women"), `Category` (manyToOne → Category, required, inverse `Subcategories`) — every subcategory belongs to exactly one category.
3. AC3: `Product` (`api::product.product`) has `Category` (manyToOne → Category, required — exactly one main category) and `Subcategories` (manyToMany → Subcategory, optional — zero or more), plus `Gender` (enumeration `Men` / `Women` / `Unisex`, optional in the schema — AC11 makes it required for Clothing and forbidden elsewhere).
4. AC4: `Product` also carries `Price` (decimal, required, min 0), `Images` (media, multiple, required, images only), `Subtitle` (string, optional), `Description` (richtext, required) and `Featured` (boolean, default false).
5. AC5: `Product.Sizes` is a required repeatable component (`product.size`, min 1: `Label`, `Stock`, optional `LengthIn`/`VolumeL`). `Product.SurfboardSpecs` is an optional single component (`product.surfboard-specs`: `SkillLevel`, `FinSetup`, `FinSetupNote`, and eight 0–100 attribute scales) — set only on surfboard products.
6. AC6: A `beforeCreate`/`beforeUpdate` lifecycle hook on `api::product.product` rejects the save with a `ValidationError` if any selected `Subcategory`'s `Category` isn't the product's own `Category`. On update, Strapi 5 sends relation *operations* (`set`/`connect`/`disconnect`), not the full list, so the hook resolves the final Category and Subcategories from the stored relations plus those operations before comparing — including a Category change checked against the Subcategories the product already has (or is being given in the same update).
7. AC7: `GET /api/products`, `/api/categories`, `/api/subcategories` (list and single) are public, no auth required. `POST`/`PUT`/`DELETE` on all three are forbidden (403) for the public role.
8. AC8: Standard Strapi filters answer both category levels and component/relation queries, with no custom endpoint: `filters[Category][Slug][$eq]=surfboards` for a main category, `filters[Subcategories][Slug][$eq]=soft-top-beginner` for a subcategory (matches if *any* of a product's Subcategories match — a product in two subcategories appears under both). Component fields (`filters[SurfboardSpecs][SkillLevel][$eq]=…`, `filters[Sizes][VolumeL][$between]=…`), `filters[Featured][$eq]=true` and `sort=Price:asc|desc` all work the same way.
9. AC9: `npm run seed:catalog` (create-only — a rerun skips anything whose `Slug` already exists, so admin edits survive) seeds 4 categories (Surfboards, Accessories, Wetsuits, Clothing), 16 subcategories under their correct category (Clothing: T-Shirts & Tanks, Shorts, Boardshorts, Tops, Swimmers — no gender duplicates; Wetsuits keeps Men's / Women's Wetsuits with NavLabel "Men" / "Women"), and all 17 products with images, required Sizes, their design Category and Subcategories, a `Gender` on every Clothing product (`samurai-pro-22-boardshort` and `boardwalk-hybrid-shorts` Men, `tidal-rash-guard` Women, `archive-team-tee` Unisex) and on no other product, and — for the 8 surfboards — SurfboardSpecs. At least one surfboard (`soft-cruiser-8-0-longboard`) has two Subcategories (`longboard` + `soft-top-beginner`); `horizon-snapback-cap` has a Category (`accessories`) and no Subcategory.
10. AC10: Reruns of `seed:catalog` create no duplicate products — `Slug` is a `uid` field, and the script's create-only check keys off it.
11. AC11: The same product lifecycle hook enforces Gender: if the product's final `Category` is Clothing (`clothing`), `Gender` is required; for any other category, `Gender` must be empty. Violations are rejected with a `ValidationError` that names the category. On update the hook resolves the final state first — the stored Category/Gender plus the request's relation operations and fields — so a Category change is checked against the Gender the product already has (moving a product out of Clothing requires clearing its Gender, moving one in requires setting it). The Subcategory check (AC6) runs first.
12. AC12: Clothing gender filters include Unisex: **Men** = `filters[Gender][$in][0]=Men&filters[Gender][$in][1]=Unisex`, **Women** = `filters[Gender][$in][0]=Women&filters[Gender][$in][1]=Unisex` (combined with `filters[Category][Slug][$eq]=clothing`). The frontend's `?gender=men|women` (site-nav, catalog-listing-page) maps to these.

## Technical Approach

**Strapi (cms/)**, built directly in the Content-Type Builder (no scaffold-then-migrate step needed — these are new content types).

- `api::category.category` (`cms/src/api/category/content-types/category/schema.json`): `Name`, `Slug` (uid), `Subcategories` (oneToMany, `mappedBy: 'Category'`), `Products` (oneToMany, `mappedBy: 'Category'`) — both inverse sides, read-only from Category.
- `api::subcategory.subcategory` (`cms/src/api/subcategory/content-types/subcategory/schema.json`): `Name`, `Slug` (uid), `Category` (manyToOne, `inversedBy: 'Subcategories'`, required), `Products` (manyToMany, `mappedBy: 'Subcategories'`, inverse side).
- `api::product.product` (`cms/src/api/product/content-types/product/schema.json`): see AC3–AC5 for the field list. `Category` is `inversedBy: 'Products'` on Category; `Subcategories` is `inversedBy: 'Products'` on Subcategory (the owning side of that manyToMany).
- Components: `product.size` (`cms/src/components/product/size.json`), `product.surfboard-specs` (`cms/src/components/product/surfboard-specs.json`).
- `cms/src/api/product/content-types/product/lifecycles.ts` implements AC6 and AC11 (`assertProductTaxonomy`; the gendered category is the constant `GENDERED_CATEGORY_SLUG = 'clothing'`, mirrored by the frontend's `GENDERED_CATEGORY_SLUGS` in `lib/routes.ts`). It normalizes every relation input shape Strapi 5 can send (a bare id/documentId, an array, or `{ set, connect, disconnect }`) into a final id list via `resolve()`, only re-checks when the incoming `data` actually touches `Category` and/or `Subcategories` (so unrelated field edits are never blocked by a stale relation snapshot), and queries stored relations one at a time rather than populating several at once — populate on a transaction's single DB connection issues its queries in parallel, which `pg` deprecates.
- `cms/src/bootstrap/catalog.ts` grants the public role `find`/`findOne` on all three UIDs (AC7), via the shared `grantPublicActions` helper in `cms/src/bootstrap/public-permissions.ts` (idempotent — same pattern as `google-login.ts`/`homepage.ts`, factored out once two specs needed it). Wired into `bootstrap()` in `cms/src/index.ts`.
- **Seed**: `cms/seed/catalog/catalog.json` is the single source of truth for categories, subcategories and products (design copy, prices, sizes, SurfboardSpecs, and per-product `CategorySlug`/`SubcategorySlugs`). `cms/scripts/seed-catalog.js` (`npm run seed:catalog`) reads it, creates anything missing by `Slug` (AC9/AC10), and uploads each product's photos from `cms/seed/catalog/<slug>/*` when present. Products without a photo folder in the JSON (`images` key) get a generated placeholder (`sharp`-rendered SVG: product name on WESTLINE ink/horizon) instead, so it's visibly obvious in the admin which products still need real photography.
- `cms/scripts/verify-catalog.mjs` (`npm run verify:catalog`) smoke-checks AC6–AC12 against a running local Strapi: public REST reachability and write-forbidding (AC7), component/relation filters and sort including the two-subcategory product appearing under both filters (AC8), full seed presence/correctness including per-product spot checks (AC9), no duplicate slugs (AC10), and the consistency hook (AC6) — exercised via the Document Service directly (the public role can't write), with every throwaway product it creates deleted again in a `finally`.

## Out of Scope

- Any frontend consumption of this API — `catalog-listing-page`, `catalog-search-filter-sort` and `product-detail-page` are separate specs.
- Draft/preview workflow (`draftAndPublish` is off, matching `homepage`).
- Localization (only the `en` locale exists).
- Product reviews, ratings, inventory beyond per-size `Stock`, or variant pricing.
- Admin-side UX beyond what the Content-Type Builder gives for free (no custom admin views).

## Standards Applied

- [global/git-workflow](../../../../../../standards/global/git-workflow.md): branch `feat/storefront-catalog/product-content-model`, commit and PR conventions.

## Changelog

| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-09-28 | Ori Chai Matan | created | Spec written to match the built model (Category + Subcategory collection types, Product relations, consistency lifecycle hook, seed, verify:catalog) — see Overview for why a flat two-level model replaced the originally-considered self-referencing Category tree. | — |
| 2026-09-29 | Ori Chai Matan | change | Added optional `Subcategory.NavLabel` (string) for site-nav's short menu labels (e.g. "Shorts" under Men, "Men" under Wetsuits; falls back to `Name`); seeded for the 8 wetsuit/clothing subcategories | 2026-09-29-site-nav |
| 2026-09-29 | Ori Chai Matan | change | Clothing taxonomy: Men's Clothing + Women's Clothing replaced by one `Clothing` category with gender-neutral subcategories (T-Shirts & Tanks, Shorts, Boardshorts, Tops, Swimmers); added `Product.Gender` (Men/Women/Unisex) — required for Clothing, forbidden elsewhere, enforced by the lifecycle hook with final-state resolution (AC11); Men/Women filters include Unisex (AC12); NavLabel kept for Wetsuits only; seed now 4 categories / 16 subcategories, `tidal-rash-guard` moved to Tops/Women so both gender filters have a dedicated product — mentor-requested: one Clothing category with gender-neutral subcategories instead of Men's/Women's Clothing duplicates | mentor review |

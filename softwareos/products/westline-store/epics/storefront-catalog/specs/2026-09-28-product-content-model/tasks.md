# Tasks — Product Content Model

> Epic: storefront-catalog
> Spec: 2026-09-28-product-content-model
> Branch: feat/storefront-catalog/product-content-model

## Backend — content types

- [x] T1 Add collection type `api::category.category` (`Name`, `Slug` uid, inverse `Subcategories`/`Products` relations) — AC1
- [x] T2 Add collection type `api::subcategory.subcategory` (`Name`, `Slug` uid, required `Category` manyToOne, inverse `Products`) — AC2
- [x] T3 Add `Category` (required manyToOne) and `Subcategories` (optional manyToMany) to `api::product.product`, alongside `Price`, `Images`, `Subtitle`, `Description`, `Featured` — AC3, AC4
- [x] T4 Add components `product.size` (repeatable, required, min 1 on Product) and `product.surfboard-specs` (single, optional on Product) — AC5

## Backend — consistency + permissions

- [x] T5 `cms/src/api/product/content-types/product/lifecycles.ts`: `beforeCreate`/`beforeUpdate` reject a save if any Subcategory's Category ≠ the product's Category; resolve Strapi 5's `set`/`connect`/`disconnect` relation operations against the stored relations first, including a Category change checked against existing Subcategories — AC6
- [x] T6 Factor the idempotent public-permission grant out of `cms/src/bootstrap/homepage.ts` into `cms/src/bootstrap/public-permissions.ts` (`grantPublicActions`); add `cms/src/bootstrap/catalog.ts` granting `find`/`findOne` on product/category/subcategory; wire `setupCatalog(strapi)` into `cms/src/index.ts` — AC7

## Backend — seed

- [x] T7 Author `cms/seed/catalog/catalog.json`: 5 categories, 17 subcategories (each with its `CategorySlug`), 17 products (each with `CategorySlug` + `SubcategorySlugs`, `Price`, `Sizes`, `SurfboardSpecs` where a board, `Featured` where set) — matches the design copy
- [x] T8 `cms/scripts/seed-catalog.js` (`npm run seed:catalog`): create-only by `Slug` for categories/subcategories/products; uploads each product's photos from `cms/seed/catalog/<slug>/*` when present, else a generated placeholder image (`sharp`) — AC9, AC10

## Verification

- [x] T9 `cms/scripts/verify-catalog.mjs` (`npm run verify:catalog`): public REST reachability + write-forbidding for all three content types (AC7); component/relation filters, `Featured`, `sort=Price` (AC8) — including that the two-subcategory product (`soft-cruiser-8-0-longboard`) appears under both its subcategory filters; full seed presence/correctness incl. per-product spot checks (AC9); no duplicate slugs (AC10); the consistency hook — mismatched create/update rejected, matching create/update accepted, Category-change-without-matching-Subcategories rejected (AC6)
- [ ] T10 Run QA against acceptance criteria (qa skill)
- [x] T11 Clothing taxonomy change: `Product.Gender` enum, hook `assertProductTaxonomy` (Subcategory + Gender rules, final-state resolution), catalog.json (4 categories, 5 Clothing subcategories, Gender on clothing), seed passes Gender, verify:catalog gains AC11 rejections + AC12 Men/Women (incl. Unisex) filters. 2026-09-29: Gender probe 11/11 and Subcategory probe 12/12 live; full `verify:catalog` pending the local DB reset

## Size model (strict, filterable sizes — replaces `Sizes`/`Label`)

- [x] T12 TDD first: `cms/scripts/verify-catalog.mjs` size-model checks (SizeType per category, matching component non-empty / other empty, Tideline BoardSizes, Samurai StandardSizes, `filters[StandardSizes][Size][$eq]=M`, `filters[BoardSizes][VolumeL][$between]`) + hook probes for rules 1–4 and stale-entry clearing; `frontend/src/__tests__/lib/strapi/sizes.test.ts` for the formatters — AC5, AC8, AC14, AC15
- [x] T13 Schema: `SizeType` enum (required), `BoardSizes` (`product.board-size`) / `StandardSizes` (`product.standard-size`) with Conditional Fields on `SizeType`; delete `product.size` — AC5, AC13
- [x] T14 `lifecycles.ts`: `assertProductSizes` (rules 1–4, final-state resolution, clear messages), called from `assertProduct` (formerly `assertProductTaxonomy`) after the Subcategory and Gender checks; `cms/src/api/product/documents/clear-stale-sizes.ts` document middleware registered in `cms/src/index.ts` — AC14, AC15
- [x] T15 Seed: `catalog.json` converted (SizeType + BoardSizes/StandardSizes, waist/XS/fins remapped), `seed-catalog.js` passes the new fields — AC9
- [x] T16 `cms/scripts/reset-catalog.js` + `npm run reset:catalog` (dev-only guards, dry run without `--yes`, seeded products only unless `--all`) — AC16
- [x] T17 `frontend/src/lib/strapi/sizes.ts`: size types + `formatBoardSize` / `formatStandardSize` — AC5
- [x] T18 Docs: `field-mapping.md`, `references.md`, `specs.index.yml` covers
- [ ] T19 Run QA against acceptance criteria (qa skill)

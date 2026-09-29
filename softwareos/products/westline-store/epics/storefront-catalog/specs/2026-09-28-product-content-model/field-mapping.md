# Field Mapping — Product Content Model

> Epic: ../../epic.md
> Spec: 2026-09-28-product-content-model

Exact Strapi field names (as built in the Content-Type Builder), for anyone writing REST queries or frontend code against this API. PascalCase throughout, matching `homepage`'s `Hero`/`BackgroundImg`/`Button` convention rather than Strapi's own lowercase defaults.

## Category — `api::category.category` (collection type)

| Field | Type | Notes |
|---|---|---|
| `Name` | string, required | e.g. `"Surfboards"` |
| `Slug` | uid (from `Name`), required | e.g. `"surfboards"` |
| `Subcategories` | relation, oneToMany → Subcategory, `mappedBy: 'Category'` | inverse side — read-only from Category |
| `Products` | relation, oneToMany → Product, `mappedBy: 'Category'` | inverse side — read-only from Category |

## Subcategory — `api::subcategory.subcategory` (collection type)

| Field | Type | Notes |
|---|---|---|
| `Name` | string, required | e.g. `"Fun Board"` |
| `Slug` | uid (from `Name`), required | e.g. `"fun-board"` |
| `Category` | relation, manyToOne → Category, `inversedBy: 'Subcategories'`, required | exactly one parent category |
| `Products` | relation, manyToMany → Product, `mappedBy: 'Subcategories'` | inverse side — read-only from Subcategory |

## Product — `api::product.product` (collection type)

| Field | Type | Notes |
|---|---|---|
| `Name` | string, required | |
| `Slug` | uid (from `Name`), required | |
| `Category` | relation, manyToOne → Category, `inversedBy: 'Products'`, required | exactly one main category |
| `Subcategories` | relation, manyToMany → Subcategory, `inversedBy: 'Products'` | zero or more; each must share the product's `Category` (enforced by the lifecycle hook — see spec.md AC6) |
| `Price` | decimal, required, min 0 | |
| `Images` | media, multiple, required, images only | |
| `Subtitle` | string | optional |
| `Description` | richtext, required | |
| `Featured` | boolean, default `false` | |
| `Sizes` | component `product.size`, repeatable, required, min 1 | |
| `SurfboardSpecs` | component `product.surfboard-specs`, single, optional | set only on surfboard products |

## Size — `product.size` (component)

| Field | Type | Notes |
|---|---|---|
| `Label` | string, required | e.g. `6'0" · 29.4L` (boards) or `36` (apparel) |
| `Stock` | integer, required, min 0, default 0 | |
| `LengthIn` | decimal | boards only |
| `VolumeL` | decimal | boards only |

## SurfboardSpecs — `product.surfboard-specs` (component)

| Field | Type | Notes |
|---|---|---|
| `SkillLevel` | enum `Beginner` / `Intermediate` / `Advanced`, required | |
| `Video` | media, single, videos only | optional |
| `FinSetup` | enum `Thruster` / `Twin` / `Quad` / `Single` / `TwoPlusOne`, required | |
| `FinSetupNote` | text | optional |
| `WaveSize`, `Break`, `Power`, `Approach`, `FootOrientation`, `Foil`, `NoseShape`, `TailWidth`, `EntryRocker`, `ExitRocker`, `RockerStyle` | integer, required, 0–100 | attribute scales |

## Filters (public REST)

| Query | Matches |
|---|---|
| `filters[Category][Slug][$eq]=surfboards` | products whose main `Category.Slug` equals `surfboards` |
| `filters[Subcategories][Slug][$eq]=soft-top-beginner` | products with `soft-top-beginner` among their `Subcategories` — any-match, so a product in two subcategories appears under both |
| `filters[SurfboardSpecs][SkillLevel][$eq]=Intermediate` | component field filter |
| `filters[Sizes][VolumeL][$between][0]=30&…[1]=40` | any Size in that range |
| `filters[Featured][$eq]=true` | |
| `sort=Price:asc` / `sort=Price:desc` | |

## Seed reference (`cms/seed/catalog/catalog.json`)

- **4 categories**: Surfboards, Accessories, Wetsuits, Clothing.
- **16 subcategories** — Surfboards: Performance Shortboard, Longboard, Fish & Twinfin, Fun Board, Soft-top / Beginner (5) · Accessories: Fins, Leashes, Traction, Travel Bags (4) · Wetsuits: Men's Wetsuits, Women's Wetsuits (2, NavLabel "Men" / "Women") · Clothing: T-Shirts & Tanks, Shorts, Boardshorts, Tops, Swimmers (5, no gender duplicates).
- **Gender** (Clothing only): the design's Clothing → Men / Women split is now `Product.Gender` — Samurai Pro + Boardwalk Hybrid Shorts Men, Tidal Rash Guard Women, Archive Team Tee Unisex. Men/Women filters include Unisex.
- **17 products**, each with a `CategorySlug` and zero or more `SubcategorySlugs`. Notably: `soft-cruiser-8-0-longboard` has two (`longboard`, `soft-top-beginner`) — the multi-subcategory case AC8/AC9 verify. `horizon-snapback-cap` has a `Category` (`accessories`) and no `SubcategorySlugs` — the no-subcategory case.

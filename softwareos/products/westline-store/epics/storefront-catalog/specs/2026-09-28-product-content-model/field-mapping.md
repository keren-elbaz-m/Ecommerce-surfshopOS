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
| `SizeType` | enum `Surfboard` / `Standard`, required | Surfboards ⇔ `Surfboard`, every other category ⇔ `Standard` (lifecycle hook, spec.md AC14) |
| `BoardSizes` | component `product.board-size`, repeatable | visible only when `SizeType` = `Surfboard` (Conditional Field); ≥1 entry required then, none otherwise (hook) |
| `StandardSizes` | component `product.standard-size`, repeatable | visible only when `SizeType` = `Standard` (Conditional Field); ≥1 entry required then, none otherwise (hook) |
| `SurfboardSpecs` | component `product.surfboard-specs`, single, optional | set only on surfboard products |

## BoardSize — `product.board-size` (component)

| Field | Type | Notes |
|---|---|---|
| `LengthFt` | integer, required, min 4, max 12 | feet, e.g. `6` |
| `LengthInches` | integer, required, min 0, max 11 | inches past the feet, e.g. `0` → 6'0"; sizes order by `LengthFt × 12 + LengthInches` |
| `VolumeL` | decimal, required | litres, e.g. `29.4` |
| `Stock` | integer, required, min 0, default 0 | |

No label field — the frontend builds `6'0" · 29.4L` (`formatBoardSize` in `frontend/src/lib/strapi/sizes.ts`). No two entries may share the same length (`LengthFt` + `LengthInches`) and `VolumeL`.

## StandardSize — `product.standard-size` (component)

| Field | Type | Notes |
|---|---|---|
| `Size` | enum `S` / `M` / `L` / `XL` / `OneSize`, required | no duplicates; `OneSize` must be the only entry when used. Shown as-is, `OneSize` as "One Size" (`formatStandardSize`) |
| `Stock` | integer, required, min 0, default 0 | |

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
| `filters[BoardSizes][VolumeL][$between][0]=30&…[1]=40` | boards with any size in that volume range |
| `filters[StandardSizes][Size][$eq]=M` | products offering size M |
| `filters[Featured][$eq]=true` | |
| `sort=Price:asc` / `sort=Price:desc` | |

## Seed reference (`cms/seed/catalog/catalog.json`)

- **4 categories**: Surfboards, Accessories, Wetsuits, Clothing.
- **16 subcategories** — Surfboards: Performance Shortboard, Longboard, Fish & Twinfin, Fun Board, Soft-top / Beginner (5) · Accessories: Fins, Leashes, Traction, Travel Bags (4) · Wetsuits: Men's Wetsuits, Women's Wetsuits (2, NavLabel "Men" / "Women") · Clothing: T-Shirts & Tanks, Shorts, Boardshorts, Tops, Swimmers (5, no gender duplicates).
- **Gender** (Clothing only): the design's Clothing → Men / Women split is now `Product.Gender` — Samurai Pro + Boardwalk Hybrid Shorts Men, Tidal Rash Guard Women, Archive Team Tee Unisex. Men/Women filters include Unisex.
- **Sizes**: the 13 surfboards are `SizeType` Surfboard with `BoardSizes`; everything else is Standard. Converted from the old free-text labels — waist 28–29 → S, 30–32 → M, 33–35 → L, 36+ → XL (stock summed); spring-suit XS → S; cap, leash and fins (`FCS II`/`Futures` summed) → `OneSize`.
- **22 products**, each with a `CategorySlug` and zero or more `SubcategorySlugs`. Notably: `soft-cruiser-8-0-longboard` has two (`longboard`, `soft-top-beginner`) — the multi-subcategory case AC8/AC9 verify. `horizon-snapback-cap` has a `Category` (`accessories`) and no `SubcategorySlugs` — the no-subcategory case.

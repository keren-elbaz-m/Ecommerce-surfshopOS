# Tasks — Home Featured Gear

> Epic: storefront-catalog
> Spec: 2026-10-14-home-featured-gear
> Branch: feat/storefront-catalog/home-featured-gear

## CMS (home-hero's homepage)

- [x] T1 Add optional `FeaturedEyebrow` / `FeaturedTitle` to `homepage/schema.json`; seed data + `seed-homepage.js` set them; update `cms/tests/homepage/seed-homepage.test.ts` (AC1)

## Frontend

- [x] T2 Failing tests: `getFeaturedProducts` (Featured filter, updatedAt desc, limit 8, card mapping, null on error) and `getHomepageFeaturedText` (AC1, AC4)
- [x] T3 Implement both (`lib/strapi/products.ts`, `lib/strapi/homepage.ts`)
- [x] T4 Failing tests: `stepCount` / `use-featured-slider`, `FeaturedSlider` (arrows, dots, aria) and `FeaturedGear` (heading + fallback, cards with hearts, null when empty) (AC1-AC4)
- [x] T5 Implement `components/home/featured-gear/` (FeaturedGear, FeaturedSlider, hook, texts, styles, barrel)
- [x] T6 Render `<FeaturedGear />` after the hero on Home; Changelog rows on home-hero (homepage fields, Home page) and catalog-listing-page (`getFeaturedProducts`) (AC1)

## Verification

- [x] T7 Full verify (dev server stopped for the build) **2026-10-14:** same run as site-footer T9 (all green).
- [x] T8 Live browser check: Home at 1280/900/375 — section copy, 6 seeded featured products newest first, arrows/dots, a heart saves; no horizontal scroll (AC1-AC5) **Done 2026-10-14** — see qa-report.md.
- [x] T9 QA against AC1-AC5 (qa skill) → qa-report.md **Run 2026-10-14:** PASS WITH NOTES, 5/5 AC — see qa-report.md.

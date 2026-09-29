# Storefront & Catalog — Tech Plan

> Epic: epic.md · Product: ../../mission.md

## Approach

Next.js pages (catalog listing, product detail) fetch product data from Strapi's REST API at request/build time; search/filter/sort are query params passed through to Strapi; Add to Cart writes to local/client cart state (no persistence yet — that's the Cart & Checkout epic's job).

## Architecture Impact

Adds new Strapi content types (Product and related types, e.g. category/attributes) for catalog data. No new services or infra — fits within the existing Next.js + Strapi + PostgreSQL architecture.

## Risks & Unknowns

- Surf gear has varied attributes per category (boards vs wetsuits vs accessories) — the Strapi content model needs to handle that without becoming unwieldy.
- Filter/sort performance and seed data volume are open questions worth a quick spike.

## Candidate Specs

- `product-content-model` — Strapi content types for Product + Category/attributes (flexible for boards/wetsuits/accessories) with seed data, exposed via REST; no frontend yet.
- `catalog-listing-page` — Next.js SSR catalog page that fetches and renders the product grid from Strapi (pagination, basic layout, category browsing); no search/filter/sort yet.
- `catalog-search-filter-sort` — adds search, category/attribute filters, and sort as query params passed through to Strapi on the listing page.
- `product-detail-page` — Next.js SSR product detail page (full info, images, attributes) fetched by slug/id from Strapi.
- `add-to-cart` — client-side cart state (add/update quantity, item count in header) wired from both listing and detail pages, no persistence/checkout.

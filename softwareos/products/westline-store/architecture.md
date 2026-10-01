# Architecture

## Components

- **Next.js app** — SSR storefront (catalog, product pages, cart, checkout, Surfboard Finder), plus customer account and seller dashboard views.
- **Strapi** — headless CMS + API layer; owns products, inventory, orders, users, and content; persists to PostgreSQL.
- **PostgreSQL** — data store, accessed via Strapi's data layer.

## Data Flow

Next.js frontend calls Strapi over REST for products, orders, users, and content. Strapi reads/writes PostgreSQL.

## Environments

TBD

## Gotchas

None known — greenfield build, nothing unusual yet.

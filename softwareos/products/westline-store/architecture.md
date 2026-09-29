# Architecture

## Components

- **Next.js app** — SSR storefront (catalog, product pages, cart, checkout, Surfboard Finder), plus customer account and seller dashboard views.
- **Strapi** — headless CMS + API layer; owns products, inventory, orders, users, and content; persists to PostgreSQL.
- **PostgreSQL** — data store, accessed via Strapi's data layer.

## Data Flow

Next.js frontend calls Strapi over REST for products, orders, users, and content. Strapi reads/writes PostgreSQL. **Not yet wired**: no fetch/API-client code in `web/` calls `cms/` today (no `NEXT_PUBLIC_STRAPI_URL` or similar), and Strapi's CORS middleware is left at its default (unconfigured allowlist) — both need setting up as part of the first integration work.

## Environments

Local dev only (`web` on Next's default port, `cms` on `0.0.0.0:1337`). No staging/prod config exists yet.

## Gotchas

- `web/src/app/page.tsx` and `layout.tsx` are still the unmodified `create-next-app` template — needs full replacement, not incremental edits.
- `cms/src/api/` and `cms/src/extensions/` are empty (`.gitkeep` only) — every content-type (Product, Order, Board, etc.) is greenfield.
- `cms/src/admin/app.example.tsx` / `vite.config.example.ts` are inactive `.example` stubs — no admin panel customization yet.
- `web/tailwind.config.ts` content globs include a `src/components/` path that doesn't exist yet — harmless, just unused.

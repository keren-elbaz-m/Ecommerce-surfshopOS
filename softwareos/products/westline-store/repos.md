# westline-store — Repos

Both apps live in the single `Ecommerce-surfshop` repo (`https://github.com/ori-chaimatan/Ecommerce-surfshop.git`), as independent top-level directories — no monorepo tooling, no shared code between them yet.

| Dir | Role | Tech |
|---|---|---|
| web | client (frontend) | Next.js 14.2 (App Router), React 18, TypeScript, Tailwind CSS |
| cms | server (backend) | Strapi 5.54, TypeScript, PostgreSQL (via `pg`) |

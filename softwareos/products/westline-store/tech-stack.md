# Tech Stack

## Frontend

- Next.js 14.2 (App Router), React 18, TypeScript
- Tailwind CSS
- Still default `create-next-app` scaffold — no pages/components built yet, no API client wired to `cms`

## Backend

- Strapi 5.54 (TypeScript)
- `@strapi/plugin-users-permissions` (installed, default config beyond JWT refresh + httpOnly sessions — gives auth/roles out of the box for customer accounts)
- `@strapi/plugin-cloud` (installed, unused)
- No content-types defined yet (`cms/src/api/`, `cms/src/extensions/` are empty)

## Database

- PostgreSQL (via `pg`, Strapi's data layer) — connection config is env-driven (`cms/config/database.ts`); defaults to sqlite if `DATABASE_CLIENT` isn't set

## Other

- No CI, no Dockerfile/deploy config, no test files yet — matches project mode `setup` (greenfield capstone build)
- `cms/config/plugins.ts` has the only real backend decisions made so far: upload MIME allow/deny list, users-permissions session config

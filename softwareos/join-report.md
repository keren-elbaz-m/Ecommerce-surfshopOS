# Join Report — WESTLINE

Generated: 2026-09-22

## Taxonomy

| Customer | Product | Repos | Shared services |
|---|---|---|---|
| westline | westline-store | `Ecommerce-surfshop` (dirs: `web/` — Next.js frontend, `cms/` — Strapi backend) | none |

SoftwareOS planning docs live in the hub repo `Ecommerce-surfshopOS` (already the case before this run — `config.yml` has no `customer_root` set, so it's assumed you run SoftwareOS commands from this hub repo). The application code lives in the separate sibling repo `Ecommerce-surfshop`.

`web` and `cms` are two independent apps inside the *same* repo (no monorepo tooling, no shared code) — not two separate repos.

## Mode

**setup** (unchanged). Shown to you: no CI, no test files, no Dockerfile/deploy config, no git tags, 1 commit ("Initial scaffold: Next.js (web) + Strapi (cms)") — matches customer/overview.md's own description of a 4-week solo capstone build. You confirmed keeping `setup`. Tests/security review/QA stay optional per spec until you run `/go-production`.

## Counts

1 product · 5 epics discovered (all already existed, pre-planned before code — none new from this scan) · 0 specs.

## Inferred vs. TBD

**Confirmed from code** (updated into existing docs):
- Repo structure: `web` (Next.js 14.2, React 18, TS, Tailwind) + `cms` (Strapi 5.54, TS, PostgreSQL via `pg`) — matches what `customer/tech-stack.md` and `products/westline-store/tech-stack.md` already said.
- `web` and `cms` have **zero wiring** between them yet — no API client in `web`, Strapi CORS left at default, no shared env var for the Strapi base URL.
- Both apps are still framework boilerplate: `web/src/app/page.tsx`/`layout.tsx` are unmodified `create-next-app`; `cms/src/api/` and `cms/src/extensions/` are empty (no content-types defined).
- One real backend decision already made: `cms/config/plugins.ts` (upload MIME allow/deny list, users-permissions JWT refresh + httpOnly sessions) — worth reusing/extending rather than rewriting.
- `@strapi/plugin-users-permissions` is installed and unconfigured beyond the above — natural fit for the `customer-account-wishlist` epic's auth.
- No payment provider, email/SMTP, storage, or analytics service is wired in yet (`cms/.env.example` only has Strapi's own secrets).

**Left as-is (already accurate from prior planning, not re-derived):** product mission, roadmap, and all 5 epic summaries/product-briefs/tech-plans — these were written before the code existed and the code doesn't yet contradict them.

**Still TBD:** success measures and key dates in `customer/overview.md`; auth method + docs for the mock payment provider in `dependencies.md`; environments beyond local dev.

## Uncertainties

- A branch `feat/customer-account-wishlist/login-signup` exists (in a linked worktree at `Ecommerce-surfshop.worktrees/login-signup`) but has no commits beyond the initial scaffold and no `specs/` folder yet — treated as an empty placeholder, not an active spec. If work has started there, run `/shape-spec` to formalize it before it drifts further from the epic docs.
- `ecosystem/ecosystem.md` was written even though this is a single-product customer (per `/join-project`'s own instructions) — `/plan-ecosystem`'s own guidance says this is normally skippable for one product. Kept it minimal; feel free to ignore or delete it later if a second product never materializes.

## What was written

- Updated: `customer/repos.md`, `products/westline-store/{mission.md, tech-stack.md, architecture.md, dependencies.md}`
- Created: `products/westline-store/repos.md`, `ecosystem/ecosystem.md`, `ecosystem/products.index.yml`, `products/westline-store/epics.index.yml`, and 5 `specs.index.yml` (one per epic, all empty)
- Unchanged: `customer/overview.md`, `customer/team.md`, `customer/tech-stack.md`, `products/westline-store/roadmap.md`, all 5 `epics/*/epic.md` + `product-brief.md` + `tech-plan.md` — already accurate
- `config.yml`: `mode: setup` confirmed unchanged

No AgentOS (`agent-os/`) folder was found in the workspace, so nothing to migrate.

## Next steps

- `/plan-epic` to refine any of the 5 existing epics further, or `/shape-spec` to start the first spec (likely `storefront-catalog` first, per the epic summaries' own stated build order).
- `/refresh-indexes` any time after manual edits to the tree.
- `/go-production` once the store is live or close to it — flips the mode and turns on the full verification gates.

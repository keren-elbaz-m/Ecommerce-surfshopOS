# References for Product Content Model

## Related Specs

- [2026-09-27-home-hero](../2026-09-27-home-hero/spec.md): same epic, first Strapi content type in the project (`homepage`). Its idempotent public-permission-grant pattern (`setupX(strapi)`, compare-before-write, wired from `cms/src/index.ts`) is directly reused here — and its own version was factored out into the shared `cms/src/bootstrap/public-permissions.ts` helper as part of this spec, so both `homepage.ts` and `catalog.ts` now call it. Its original (later-dropped) seed step — uploading images through `strapi.plugin('upload').service('upload')` on Strapi boot — is also the precedent for `seed-catalog.js`'s image upload, run as a standalone script instead of at boot since this seed is deliberately not automatic (see spec.md's Technical Approach).

## Upstream Meeting

_None. No meeting write-ups exist yet._

## Similar Implementations

### Idempotent Strapi bootstrap + public permission grant

- **Location:** `cms/src/bootstrap/public-permissions.ts` (`grantPublicActions`), used by `cms/src/bootstrap/catalog.ts` and `cms/src/bootstrap/homepage.ts`
- **Relevance:** the `find`/`findOne` public grant for product/category/subcategory (AC7) follows the exact pattern established for `homepage` — compare existing permissions before writing, so reboots are no-ops.
- **Key patterns:** `export async function setupX(strapi)`, `strapi.log.warn` on misconfiguration (missing public role), wired into `bootstrap()` in `cms/src/index.ts` alongside the other `setup*` calls.

### Strapi 5 relation-update semantics

- **Location:** `cms/src/api/product/content-types/product/lifecycles.ts`
- **Relevance:** Strapi 5's Document Service sends relation *operations* (`set`/`connect`/`disconnect`) on update rather than a full replacement list — the AC6 consistency hook has to resolve the final relation state from the stored data plus those operations before it can validate anything. No prior code in this repo did this; it's new ground for the project.
- **Key patterns:** normalize every input shape (bare id/documentId, array, or `{set,connect,disconnect}`) into one `resolve()` call; query one relation at a time inside `beforeCreate`/`beforeUpdate` rather than populating several at once (populate on a transaction's single DB connection runs its sub-queries in parallel, which `pg` deprecates).

### create-only seed script run out-of-process

- **Location:** `cms/scripts/seed-catalog.js`, `cms/scripts/verify-catalog.mjs`
- **Relevance:** neither runs at Strapi boot (unlike `homepage`'s original seed attempt) — both are standalone scripts using `createStrapi(await compileStrapi()).load()` / `.destroy()`, safe to run against a live `strapi develop` since they talk to the same Postgres database.
- **Key patterns:** `strapi.documents(uid).findFirst({ filters: { Slug } })` before every create (idempotent reruns); explicit `process.exit(0)` after `.then()` — Strapi leaves background timers running after `destroy()`, and one firing after the DB pool is gone crashes the process with a Knex timeout even though the script already finished its work.

## Size model (2026-10-01)

- **[Related]** [2026-09-29-catalog-listing-page](../2026-09-29-catalog-listing-page/spec.md) — owns `frontend/src/lib/strapi/products.ts`, deliberately left unchanged: the listing doesn't fetch sizes. The size types and formatters live in `frontend/src/lib/strapi/sizes.ts` (this spec) for product-detail-page / surfboard-finder to import.
- **Conditional Fields (Strapi 5.54.0)** — `@strapi/types/dist/schema/attribute/base.d.ts` (`conditions.visible`, JSON Logic); admin drops hidden fields from the save payload (`content-manager/dist/admin/pages/EditView/utils/data.mjs`, `collectInvisibleAttributes`); the server entity validator skips all validation for a hidden field (`core/dist/services/entity-validator/index.mjs`) — why the size rules live in the lifecycle hook and stale entries are cleared by a document middleware.

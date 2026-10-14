# References for Home Hero

## Related Specs

- **[Considered]** [2026-09-22-account-signup-login](../../../customer-account-wishlist/specs/2026-09-22-account-signup-login/spec.md): matched on `/` (a route prefix of `/login`, `/signup`, `/forgot-password`, `/reset-password`, `/connect/google/redirect`). Dismissed in Step 4d: those are the auth pages, a different concern from the Home hero. The match comes only from `/` being the root route.

## Upstream Meeting

_None. No meeting write-ups exist yet._

## Similar Implementations

### Server-side Strapi fetch

- **Location:** `frontend/src/app/api/auth/*/route.ts`, `frontend/src/lib/auth/session.ts`
- **Relevance:** the only existing code that talks to Strapi.
- **Key patterns:** `const STRAPI_URL = process.env.STRAPI_URL ?? 'http://localhost:1337'`, which is server-only (no `NEXT_PUBLIC_`). Reuse the same env var and default in `frontend/src/lib/strapi/homepage.ts`.

### Idempotent Strapi bootstrap

- **Location:** `cms/src/bootstrap/google-login.ts`, wired from `cms/src/index.ts`
- **Relevance:** the hero's public-permission grant and seed-if-empty run at bootstrap too.
- **Key patterns:** export a `setupX(strapi)` function, compare before writing so reboots are no-ops, and `strapi.log.warn` for misconfiguration.

### Home placeholder + Tailwind motion

- **Location:** `frontend/src/app/[locale]/[[...segments]]/page.tsx` (`Waves`, Coming Soon block), `frontend/tailwind.config.ts`
- **Relevance:** it becomes the hero fallback (AC13) and shows the project's motion conventions.
- **Key patterns:** `motion-reduce:animate-none`, custom keyframes in `tailwind.config.ts`, tokens `ink` / `horizon` / `horizon-deep` / `horizon-ink`, `font-display` (Poppins).

### Test conventions

- **Location:** `frontend/src/__tests__/**`, `frontend/vitest.config.ts`
- **Relevance:** TDD for the fetcher and the carousel.
- **Key patterns:** a central `__tests__` tree that mirrors `src/`, Vitest globals + jsdom + RTL, and the `@/` alias.

## Visuals

- `visuals/home.html` is a copy of the design export `westline-site-desing/index.html` (self-contained, images inlined as base64). The live, editable version is https://claude.ai/artifact/6WjmQQRaM6vSWmpBuZDupW
- `visuals/home-hero-notes.md` has the hero section's CSS, markup and carousel behavior, with base64 stripped.

# References for Site Footer

## Related Specs

- **[Related]** [2026-09-27-home-hero](../2026-09-27-home-hero/spec.md): the pattern this follows (single type, public `find` via bootstrap, create-only seed, `strapiFetch` with a fallback).
- **[Related]** [2026-09-29-site-nav](../2026-09-29-site-nav/spec.md): the header counterpart; the footer's wordmark matches the nav's.
- **[Related]** [customer-account-wishlist / 2026-09-22-account-signup-login](../../../customer-account-wishlist/specs/2026-09-22-account-signup-login/spec.md): owns `app/[locale]/layout.tsx` and `cms/src/index.ts` (one line each).
- **[Related]** [customer-account-wishlist / 2026-10-13-account-profile](../../../customer-account-wishlist/specs/2026-10-13-account-profile/spec.md): its footer bar (AC14) is replaced by this footer.

## Similar Implementations

- `cms/src/api/homepage/`, `cms/src/bootstrap/homepage.ts`, `cms/scripts/seed-homepage.js`, `cms/tests/homepage/seed-homepage.test.ts`: the single type, permission and seed pattern.
- `frontend/src/lib/strapi/homepage.ts` + `components/home/hero/Hero.tsx`: fetch, normalise and fall back.

## Visuals

- `visuals/footer-notes.md`

# References for Site Nav

## Related Specs

- **[Related]** [2026-09-22-account-signup-login](../../../customer-account-wishlist/specs/2026-09-22-account-signup-login/spec.md) owned the header before this spec. It keeps its AC8 (the account icon reflects logged-in vs logged-out state) and `frontend/src/components/logout-button.tsx`, and it hands `frontend/src/components/site-nav.tsx` over to this spec. It was found by referral in Step 4d: its Covers still listed the pre-rename `web/…` paths, so the automatic search missed it. Its Covers was backfilled as part of this spec's Task 1.
- **[Related]** [2026-09-28-product-content-model](../2026-09-28-product-content-model/spec.md) is the data source (Category + Subcategory). This spec adds the optional `Subcategory.NavLabel`, recorded in that spec's Changelog.
- **[Related]** catalog-listing-page (not shaped yet) must serve this spec's URL scheme and reuse `NAV_GROUPS`:
  - `/products/category/<categorySlug>`
  - `/products/category/<categorySlug>?sub=<subcategorySlug>`
  - `/products/category/clothing`, a virtual group

  The product slugs `category` and `clothing` must stay reserved, because product-card links products at `/products/<slug>`.

## Upstream Meeting

_None._

## Similar Implementations

### Current header

- **Location:** `frontend/src/components/site-nav.tsx` (replaced), `frontend/src/components/logout-button.tsx` (reused)
- **Key patterns:** a server component calls `getSession()` (`frontend/src/lib/auth/session.ts`). Logged out shows a Sign In link to `/auth/login`; logged in shows `<LogoutButton/>`. It's sticky and white with a bottom border, 64px tall.

### Strapi fetch

- **Location:** `frontend/src/lib/strapi/homepage.ts`
- **Key patterns:** a server-only fetch with `next: { revalidate: 60 }`, which logs and returns `[]` on error, so the page never breaks.

### Client islands + tests

- **Location:** `frontend/src/components/home/hero/hero-carousel.tsx` and its tests
- **Key patterns:** fake timers for delays, a server shell with small `'use client'` parts, and tests mocking the data layer like `hero.test.tsx`.

## Visuals

- `westline-workplace/westline-site-desing/index.html`, referenced by path and not copied. `visuals/nav-notes.md` has the extracted CSS, markup and JS.

# Site Footer

> Epic: ../../epic.md
> Status: shaped
> Branch: feat/storefront-catalog/site-footer
> Covers: cms/src/api/footer/**, cms/src/components/shared/link.json, cms/src/components/shared/footer-column.json, cms/src/bootstrap/footer.ts, cms/scripts/seed-footer.js, cms/seed/footer/**, cms/tests/footer/**, frontend/src/components/site-footer/**, frontend/src/lib/strapi/footer.ts, frontend/src/__tests__/components/site-footer/**, frontend/src/__tests__/lib/strapi/footer.test.ts

## Overview

A site-wide footer, dynamic in Strapi like the Home hero: a `footer` single type holds the tagline, link columns, legal links and copyright. Next.js renders it server-side at the end of every page. Design: `westline-site-desing/index.html` `footer` (see visuals/footer-notes.md). There's no tech-plan candidate for it: it's site chrome, like site-nav. Ori asked for it on 2026-10-14.

Decisions (2026-10-14):
- **Every page**, including My Account, whose minimal bar (profile.html) is removed so there's one footer. account-profile AC14 is superseded.
- **The seed contains only links to pages that exist.** The design's other links (New Arrivals, Shipping & Returns, Size Guide, Contact, Privacy, Terms, Seller Login, Surfboard Finder) are added in the admin once their pages exist.

## Goals & User Stories

- As a shopper, every page ends with a footer that tells me what WESTLINE is and links me to the main shopping and account pages.
- As a content editor, I can change the footer's tagline, columns, links and copyright in Strapi without a deploy.

## Acceptance Criteria

1. AC1: Every page ends with the footer, rendered server-side from the Strapi `footer` single type: brand (WESTLINE + tagline), link columns in order (title + links), and a bottom bar (copyright + legal links).
2. AC2: `GET /api/footer` is public and read-only (public `find` granted on every boot, like the homepage).
3. AC3: `npm run seed:footer` (in `cms/`, create-only) creates the footer when none exists:
   - **Tagline:** the design's tagline.
   - **Shop:** Surfboards → `/products/category/surfboards`, Wetsuits → `/products/category/wetsuits`, Apparel → `/products/category/clothing`.
   - **Account:** Sign In → `/auth/login`, Create Account → `/auth/register`, Wishlist → `/account#wishlist`, Order History → `/account#orders`.
   - **No Help column and no legal links:** none of their target pages exist yet.
   - **Copyright:** "© 2026 WESTLINE. Find Your Line."
   - An existing footer is never changed.
4. AC4: Internal links (`/…`) use `next/link`, and `http(s)://` links are plain anchors. Anything else (e.g. `javascript:`, `//host`) is dropped, and a column left with no valid links isn't shown.
5. AC5: If Strapi is down or there's no footer document, the page still renders, with a minimal bar ("WESTLINE" + "© 2026 WESTLINE. Find Your Line."), and never crashes.
6. AC6: Layout matches the design at 1280, 760 and 375 (4 columns, then 2 columns at ≤760px), with no page-level horizontal scroll. Strapi edits appear within 60 seconds.
7. AC7: My Account no longer renders its own footer bar.

## Technical Approach

**CMS:**
- Components:
  - `shared.link`: `Label` and `Url`, both required strings.
  - `shared.footer-column`: `Title` (required) and `Links` (repeatable `shared.link`).
- Single type `api::footer.footer` (`draftAndPublish: false`): `Tagline` text, `Columns` (repeatable `shared.footer-column`), `LegalLinks` (repeatable `shared.link`), `Copyright` string. Its router, controller and service use `factories.createCore*`, like `homepage`.
- `bootstrap/footer.ts` calls `grantPublicActions(['api::footer.footer.find'])`. It's registered in `src/index.ts` (account-signup-login's file; one line).
- `scripts/seed-footer.js` + `seed/footer/footer.json`, mirroring `seed-homepage.js`: create-only, a production guard and testable exports. Run with `npm run seed:footer`.

**Frontend:**
- `lib/strapi/footer.ts`: `getFooter()` via `strapiFetch` (populate `Columns.Links` and `LegalLinks`; a 404 is silent). It normalises to `{ tagline, columns[{ title, links[{ label, href, external }] }], legalLinks, copyright }`, drops unsafe hrefs (`isSafeHref`: `/` but not `//`, or `http(s)://`) and empty columns, and returns `null` when the footer is unavailable.
- `components/site-footer/`: `SiteFooter.tsx` (async server component, one return; it chooses between the full footer and the fallback), `site-footer-texts.ts` (wordmark, fallback copyright, "Footer" label), `site-footer.styles.ts`, `index.ts`.
- `app/[locale]/layout.tsx`: renders `<SiteFooter />` after `{children}` (account-signup-login's file).
- `components/account/AccountPage.tsx`: the footer bar and its strings and styles are removed (account-profile's files).

## Out of Scope

- The design's links to pages that don't exist yet (Help column, legal links, Seller Login).
- A newsletter section.
- A mobile accordion (not in the design).

## Standards Applied

- [frontend/components](../../../../../../standards/frontend/components.md): one component and one return per file; texts file; Strapi via `strapiFetch`; URLs from Strapi are validated.
- [global/git-workflow](../../../../../../standards/global/git-workflow.md): branch `feat/storefront-catalog/site-footer`.
- [global/tech-stack](../../../../../../standards/global/tech-stack.md): no new dependencies.

## Changelog

| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-10-14 | Ori Chai Matan | created | Initial shaping | — |

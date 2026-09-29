# Tasks — Site Nav

> Epic: storefront-catalog
> Spec: 2026-09-29-site-nav
> Branch: feat/storefront-catalog/site-nav

## Strapi

- [x] T1 Add optional `NavLabel` (string) to `cms/src/api/subcategory/content-types/subcategory/schema.json`; add NavLabels to the 8 wetsuit/clothing subcategories in `cms/seed/catalog/catalog.json` and pass it through in `cms/scripts/seed-catalog.js`
- [~] T2 Backfill NavLabel on the local DB (one-off, empty-only, not committed); confirm via `GET /api/categories?populate[Subcategories]…` and that `npm run verify:catalog` still passes. **2026-09-29:** backfill set 8 NavLabels (0 kept, 0 missing), REST shows them. `verify:catalog` has 1 FAIL unrelated to NavLabel: Samurai Pro now has 5 photos, not 6 (product last saved 2026-09-28 14:01, before this work — likely an image removed in the admin); every other check passes

## Data + menu model

- [x] T3 Write failing tests `frontend/src/__tests__/lib/strapi/navigation.test.ts` (query incl. sort + NavLabel field, revalidate 60, [] on network error / non-2xx)
- [x] T4 Implement `frontend/src/lib/strapi/navigation.ts` (`getNavigationCategories`) to pass T3
- [x] T5 Write failing tests `frontend/src/__tests__/components/site-nav/nav-menu.test.ts` (Strapi order kept; mens/womens-clothing → one Clothing item at the first one's position with Men/Women columns; "All <Name>" first link; NavLabel ?? Name; URL builders incl. `?sub=` and `/products/category/clothing`; group omitted when its categories are missing)
- [x] T6 Implement `frontend/src/components/site-nav/nav-menu.ts` (`NAV_GROUPS`, `categoryHref`, `subcategoryHref`, `buildNavMenu`) to pass T5

## UI

- [x] T7 Write failing tests `frontend/src/__tests__/components/site-nav/SiteNav.test.tsx` (AC3, AC6–AC9, AC11: mega-menu hover open + 250ms close, focus-within, Escape closes the open mega-menu and returns focus to its top-level link, mobile toggle aria-expanded, accordion aria-expanded + labels, link click closes panel, Sign In vs LogoutButton, Wishlist/Cart hrefs, no search/cart count/Finder, empty menu still renders icons)
- [x] T8 Implement `DesktopMenu.tsx` and `MobileMenu.tsx` (client islands) in `frontend/src/components/site-nav/`
- [x] T9 Implement `SiteNav.tsx` (server shell) + `index.ts`; delete `frontend/src/components/site-nav.tsx`; add JetBrains Mono to the font link in `frontend/src/app/[locale]/layout.tsx` + `fontFamily.mono` in `tailwind.config.ts`

## Verification

- [x] T10 Run the full frontend suite (testing skill), `npm run lint`, `tsc --noEmit`; hero still sized to the 64px header. **2026-09-29:** 114/115 (only the pre-existing hero AC13 failure); lint + `tsc` clean; `/` and `/auth/login` return 200 with the Strapi menu server-rendered (4 items, NavLabel wording, correct hrefs), header `h-16`, hero still `calc(100vh-64px)`; min-[861px] breakpoint classes generated
- [ ] T11 Run QA against acceptance criteria (qa skill) incl. a browser check at desktop and mobile widths against the design, keyboard-only mega-menu access, logged-in vs logged-out icons

## Tidy-up (frontend/components standard)

- [x] T12 Tidy-up: `lib/strapi/client.ts`, `lib/routes.ts`, `shared/components/icons/Icon.tsx`; DesktopMenu single open item + timer, MobileMenu `Set`, inline sub-markup; new tests for client, routes, one-open-at-a-time, focus-keeps-open, accordion reset; 2026-09-29: 124/125 (pre-existing hero AC13 only), tsc + lint clean, live header HTML identical apart from `aria-hidden` on the account icon
- [x] T13 Add `site-nav-texts.ts` and use it in SiteNav, DesktopMenu, MobileMenu, nav-menu.ts. 2026-09-29: 124/125 (pre-existing hero AC13 only), tsc + lint clean, live header HTML byte-identical

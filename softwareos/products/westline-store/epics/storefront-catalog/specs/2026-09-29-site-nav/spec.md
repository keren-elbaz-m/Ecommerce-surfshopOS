# Site Nav

> Epic: ../../epic.md
> Status: in-progress
> Branch: feat/storefront-catalog/site-nav
> Covers: frontend/src/components/site-nav/**, frontend/src/lib/strapi/navigation.ts, frontend/src/lib/strapi/client.ts, frontend/src/lib/routes.ts, frontend/src/shared/components/icons/**, frontend/src/__tests__/components/site-nav/**, frontend/src/__tests__/lib/strapi/navigation.test.ts, frontend/src/__tests__/lib/strapi/client.test.ts, frontend/src/__tests__/lib/routes.test.ts

## Overview

Replaces the minimal header (wordmark + Sign In / Log Out) with the design's site header:
- category links, each opening a mega-menu
- Wishlist and Cart icons
- a mobile hamburger with a slide-in accordion panel

The design source is `westline-site-desing/index.html` (`.site-nav`, `.mega-menu`, `.nav-icons`, `.mobile-panel`). The menu is built server-side from Strapi's Category and Subcategory (product-content-model), with no hardcoded subcategory lists.

The header is the catalog's main entry point, so this spec also fixes the **category URL scheme** that catalog-listing-page must serve. It's not in the tech plan's candidate list.

The account icon's auth behavior is owned by account-signup-login (its AC8) and stays unchanged. That spec hands the header file over to this one.

## Goals & User Stories

- As a shopper on desktop, I hover or tab to a category in the header and see its subcategories in a mega-menu. One click takes me to that category or subcategory.
- As a shopper on mobile, I open the menu with the hamburger and expand each category as an accordion.
- As a shopper, I reach Sign In / Log Out, my wishlist and my cart from the header on every page.
- As a content editor, when I add or rename a category or subcategory in Strapi, the menu follows it without a deploy. I can set a short menu label (`NavLabel`) when the full name reads badly in a column.

## Acceptance Criteria

1. AC1: The header is server-rendered from Strapi Categories with their Subcategories, both in Strapi order (`sort=id`). No subcategory list is hardcoded in the frontend.
2. AC2: The menu items are the Strapi categories in order. The two categories in the `NAV_GROUPS` constant, `mens-clothing` and `womens-clothing`, are collapsed into **one** item, **Clothing**, placed where the first of them appears. With today's data that gives Surfboards, Accessories, Wetsuits, Clothing.
3. AC3: A regular item's mega-menu is one column: an "All <Category Name>" link first (bold, with a bottom border), then one link per subcategory. Clothing's mega-menu has two columns headed **Men** and **Women**, each listing that category's subcategories. The column heads use the design's style and link to their category.
4. AC4: The subcategory link text is `NavLabel` when set, otherwise `Name`. `NavLabel` is a new optional string field on Strapi `Subcategory`.
5. AC5: URLs:
   - Category: `/products/category/<categorySlug>`.
   - Subcategory: `/products/category/<categorySlug>?sub=<subcategorySlug>`.
   - Clothing: `/products/category/clothing`. `clothing` is a virtual group that `NAV_GROUPS` resolves to `mens-clothing` + `womens-clothing`; `NAV_GROUPS` and the URL builders live in `frontend/src/lib/routes.ts` for catalog-listing-page to reuse.
   - Every link uses `next/link`, and they all 404 until catalog-listing-page, which is accepted.
6. AC6: Desktop (above 860px): links sit inline next to the wordmark. A mega-menu opens on hover and on keyboard focus within its item, and closes 250ms after the pointer leaves, bridging the gap between the link and the panel, as in the design. Pressing **Escape** closes an open mega-menu and returns focus to its top-level link. The panel spans the full width directly under the header.
7. AC7: Mobile (860px and below): the inline links are hidden and a hamburger (`aria-label="Menu"`, `aria-expanded`) shows.
   - It toggles a full-height ink panel below the header, with one row per menu item: a link plus a chevron toggle (`aria-expanded`, `aria-label="Toggle <Item> submenu"`) that expands its sub-links. Clothing's sub-links are grouped under Men and Women heads.
   - Tapping any link in the panel closes it.
8. AC8: Icons: Wishlist → `/account#wishlist` (`aria-label="Wishlist"`) and Cart → `/cart` (`aria-label="Cart"`), as plain links. The account icon keeps today's behavior: logged out shows a Sign In link to `/auth/login`, logged in shows the existing `LogoutButton` (account-signup-login AC8).
9. AC9: Hidden, with no dead UI: the Finder promo inside mega-menus, the "Surfboard Finder" link (desktop and mobile), the search button and the cart count don't render.
10. AC10: Visuals match the design's solid header: white, 1px bottom border, sticky, **64px tall** (the hero depends on it), and a 22px Poppins 800 wordmark linking to `/`.
    - Nav links: 13.5px, 600 weight, uppercase, 0.04em tracking, 28px gap, horizon on hover.
    - Mega-menu: white, top border, shadow, 36/40px vertical padding, 56px column gap, 14.5px links.
    - Column heads: JetBrains Mono 11px uppercase, muted.
    - Mobile panel: ink background, white uppercase links, and sub-links at 14px.
11. AC11: If Strapi is unreachable or errors, the header still renders (wordmark and icons) with no menu items. The page never fails because of the nav.

## Technical Approach

**Data: `frontend/src/lib/strapi/navigation.ts`**
- `getNavigationCategories(): Promise<StrapiNavCategory[]>`, server-only, runs:
  `GET /api/categories?fields[0]=Name&fields[1]=Slug&sort[0]=id:asc&populate[Subcategories][fields][0]=Name&populate[Subcategories][fields][1]=Slug&populate[Subcategories][fields][2]=NavLabel&populate[Subcategories][sort][0]=id:asc&pagination[pageSize]=100`
  - Fetched through `strapiFetch` (`frontend/src/lib/strapi/client.ts`: `STRAPI_URL` + the shared server fetch helper, default `revalidate: 60`), which logs and returns the fallback on a network error, non-2xx or unparsable body. `navigation.ts` returns `[]` then (AC11). The query was verified live on 2026-09-29.

**Menu model: `frontend/src/components/site-nav/nav-menu.ts`** (pure TS)
- Imports `NAV_GROUPS`, `categoryHref(slug)` and `subcategoryHref(categorySlug, subSlug)` from **`frontend/src/lib/routes.ts`**, the single routes file (also `productHref` and the fixed `HOME_HREF` / `LOGIN_HREF` / `WISHLIST_HREF` / `CART_HREF`).
- `NAV_GROUPS = [{ slug: 'clothing', label: 'Clothing', columns: [{ label: 'Men', categorySlug: 'mens-clothing' }, { label: 'Women', categorySlug: 'womens-clothing' }] }]`
- `buildNavMenu(categories)` returns `NavItem[]`:
  - `{ label, href, columns: { head?: { label, href }, allLink?: { label, href }, links: { label, href }[] }[] }`
  - It applies the grouping (AC2), the "All …" link (AC3), `NavLabel ?? Name` (AC4) and the URLs (AC5).
  - A group only appears if at least one of its categories exists.

**Components: `frontend/src/components/site-nav/`**
- `SiteNav.tsx` (server): calls `getSession()` and `getNavigationCategories()`, then `buildNavMenu`. It renders:
  - the `<header className="sticky top-0 z-50 h-16 border-b border-border bg-white">` shell
  - the wordmark
  - `<DesktopMenu items/>`
  - the icons (account via the existing Sign In link / `@/components/logout-button`, Wishlist, Cart)
  - `<MobileMenu items/>`
- `DesktopMenu.tsx` (`'use client'`): **one** open item and **one** 250ms close timer for the whole menu. The open item is derived from the hovered href, the focused href (focus within an item) and the Escape-dismissed href. Only one mega-menu is open at a time: entering another item switches over immediately. Items, columns and links are rendered inline in the maps (no `DesktopMenuItem` / `MegaColumn`). The panel is `absolute inset-x-0 top-full` below the header.
  - A keydown handler on each item: on Escape it closes the panel and focuses the item's top-level link. The dismissal holds until the pointer re-enters or focus leaves the item.
- `MobileMenu.tsx` (`'use client'`): the hamburger button with `aria-expanded`, and the panel `fixed inset-x-0 top-16 bottom-0 overflow-y-auto bg-ink`. The expanded accordions are a `Set` of item hrefs in MobileMenu's own state, cleared whenever the panel closes (so it always reopens collapsed). Groups are rendered inline in the map (no `MobileGroup`), the hamburger SVG is a `HAMBURGER` JSX constant, and link clicks close the panel.
- `index.ts` exports `SiteNav`, keeping `import { SiteNav } from '@/components/site-nav'` in `layout.tsx` working.
- UI strings live in `site-nav-texts.ts` (`texts`): the wordmark, Sign In, the aria-labels (Account, Wishlist, Cart, Main, Menu, Mobile), and the functions `allCategory(name)` ("All <name>") and `toggleSubmenu(item)` ("Toggle <item> submenu"). `SiteNav`, `DesktopMenu`, `MobileMenu` and `nav-menu.ts` import it. Strapi category/subcategory names and the `NAV_GROUPS` labels (Clothing/Men/Women, in `lib/routes.ts`) aren't texts.
- Account, Wishlist and Cart icons are `<Icon name="account" | "heart" | "cart" />` from **`frontend/src/shared/components/icons/Icon.tsx`**, the single icon component with an internal map of the design's SVG shapes (also `arrow-left` / `arrow-right`). URLs come from `lib/routes.ts`. Tokens come from `tailwind.config.ts`.
- **JetBrains Mono** (the column heads) isn't loaded yet. It's added to the Google Fonts `<link>` in `frontend/src/app/[locale]/layout.tsx` (owned by account-signup-login; a one-line edit), plus a `fontFamily.mono` token in `tailwind.config.ts`.

**Strapi (in product-content-model's area)**
- `cms/src/api/subcategory/content-types/subcategory/schema.json`: add `"NavLabel": { "type": "string" }` (optional).
- `cms/seed/catalog/catalog.json`: add `NavLabel` for Men's/Women's Wetsuits ("Men" / "Women") and the six clothing subcategories ("T-Shirts & Tanks", "Shorts", "Boardshorts", "Tops", "Shorts", "Swimmers"). `seed-catalog.js` passes it through.
- The seed is create-only, so the **local DB is backfilled** by a one-off script in my scratchpad. It sets NavLabel only where it's empty, and I'll show you the output. `verify-catalog.mjs` isn't changed.

**Removed:** `frontend/src/components/site-nav.tsx`, replaced by the folder.

**Tests (TDD):**
- `__tests__/lib/strapi/navigation.test.ts` (node): query, revalidate, `[]` on error.
- `__tests__/components/site-nav/nav-menu.test.ts` (node): AC2–AC5.
- `__tests__/components/site-nav/SiteNav.test.tsx` (jsdom + RTL, with the fetch/session mocked like `hero.test.tsx` does):
  - AC3 and AC6–AC9: mega-menu hover + 250ms close with fake timers, focus-within, the mobile toggle, the accordion `aria-expanded`, the panel closing on link click
  - the auth states
  - hidden items absent
  - AC11: an empty menu still renders the icons

## Out of Scope

- The transparent-over-hero header variant and scroll-to-solid behavior. The header is always solid.
- Search, cart count, favorites/wishlist logic, the Surfboard Finder link and the mega-menu Finder promo.
- The pages behind the links: catalog-listing-page (`/products/category/…`), `/account`, `/cart`.
- Kids and Gear entries (not in Strapi) and a skip-to-content link.
- Changing the auth behavior itself (account-signup-login).
- Editor-controlled menu order. Items and subcategories follow Strapi `id` (creation order), so editors can't reorder without recreating entries. A Category (or Subcategory) `Order` field would be a later change.

## Standards Applied

- [global/git-workflow](../../../../../../standards/global/git-workflow.md): branch `feat/storefront-catalog/site-nav`, commit and PR conventions.
- [frontend/components](../../../../../../standards/frontend/components.md): one component and one return per file, PascalCase components / kebab-case TS, all Strapi fetches via `lib/strapi/client.ts`, all URLs via `lib/routes.ts`.

## Changelog

| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-09-29 | Ori Chai Matan | created | Initial shaping | — |
| 2026-09-29 | Ori Chai Matan | change | Tidy-up to the new frontend/components standard (written in this change): added `lib/strapi/client.ts` (`STRAPI_URL` + `strapiFetch`, used by navigation/homepage/media), `lib/routes.ts` (`productHref`, `categoryHref`, `subcategoryHref`, `NAV_GROUPS`, fixed site URLs — moved out of `nav-menu.ts`), and `shared/components/icons/Icon.tsx`; DesktopMenu now holds one open item + one close timer (only one mega-menu open at a time) and MobileMenu a `Set` of expanded hrefs, with sub-markup inlined; Extended covers with frontend/src/lib/strapi/client.ts, frontend/src/lib/routes.ts, frontend/src/shared/components/icons/** and their tests. No tested behavior changed; header HTML identical except `aria-hidden` on the decorative account icon | frontend/components standard |
| 2026-09-29 | Ori Chai Matan | change | Moved all UI strings into `components/site-nav/site-nav-texts.ts` (interpolated strings as functions: `allCategory`, `toggleSubmenu`); rendered text and aria attributes unchanged (live header HTML byte-identical) | frontend/components standard |

# Home Hero

> Epic: ../../epic.md
> Status: in-progress
> Branch: feat/storefront-catalog/home-hero
> Covers: /, frontend/src/components/home/hero/**, frontend/src/lib/strapi/homepage.ts, cms/src/api/homepage/**, cms/src/components/home/hero-slide.json, cms/src/bootstrap/homepage.ts, cms/seed/home-hero/**, frontend/src/__tests__/components/home/hero/**, frontend/src/__tests__/lib/strapi/homepage.test.ts

## Overview

Replaces the Home page's "Coming Soon" placeholder with the design's full-bleed hero carousel. Each slide has a headline, an optional subtext, a background image and a primary CTA. All slide content comes from a new Strapi single type `homepage` and is rendered server-side in Next.js. This is the storefront's entry point: its CTAs funnel shoppers into the catalog. It is also the first Strapi content type in the project, so it proves the Next.js → Strapi content pipeline (content type, public permission, seed, fetch, `next/image`) end to end. It is **not** in the tech plan's Candidate Specs list; it was added as the epic's front door. It serves the success criterion ("a shopper can land on the catalog…") indirectly.

## Goals & User Stories

- As a shopper landing on `/`, I see a striking, on-brand hero that tells me what WESTLINE is and gives me one clear next step (shop, take the Finder, see a collection).
- As a shopper, I can move between hero slides with arrows, dots or a swipe, or just let them rotate.
- As a content editor, I can change hero headlines, subtexts, images and CTAs, add or remove slides, and reorder them in the Strapi admin without a code deploy.

## Acceptance Criteria

1. AC1: `/` renders a hero carousel whose slides (headline, optional subtext, background image, CTA label + link), in order, come from the Strapi `homepage` single type. No hero copy or image is hardcoded in the frontend.
2. AC2: The hero is server-rendered: every slide's headline and CTA are present in the initial HTML response, before JavaScript runs.
3. AC3: A fresh local Strapi seeds the three design slides ("Westline" → Shop Now, "Find your Surfboard" → Take the Finder, "The Dawn Patrol Collection" → Shop the Collection) with their design images when `homepage` is empty. Seeding never overwrites existing content.
4. AC4: The `homepage` content is publicly readable (`GET /api/homepage` works without auth) and read-only for the public role.
5. AC5: Visuals match the design's hero section. The hero fills the viewport below the site nav. Each slide shows a cover-fit background image, a left-to-right dark scrim, and a bottom-left white headline (Poppins display, `clamp(28px, 4.2vw, 56px)`). The optional subtext sits under the headline. The primary button uses horizon blue with a horizon-deep hover, uppercase Inter 700. Arrows sit mid-left and mid-right (44px, 36px at ≤640px), and dots sit bottom-center.
6. AC6: The active slide gets the slow zoom (to 1.08 over ~11s) and a slide-in of its text. Transitions between slides take ~0.3s, and the carousel loops continuously in both directions (last → first animates forward, not a jump back).
7. AC7: Next/prev arrows and dots change the slide. Dots mark the active slide with `aria-current="true"`. Any interaction restarts the autoplay timer.
8. AC8: The carousel auto-advances every 5s. It pauses while hovered (only on devices with real hover, i.e. `(hover: hover) and (pointer: fine)`) and while focus is inside it, and resumes on leave or blur.
9. AC9: On touch devices, a horizontal swipe of ≥40px moves to the next or previous slide. A mostly vertical drag still scrolls the page.
10. AC10: With `prefers-reduced-motion: reduce`, there is no autoplay and no zoom or slide animation. Arrows, dots and swipe still work.
11. AC11: Accessibility: the hero is a `section` with `aria-roledescription="carousel"` and `aria-label="Featured"`. Inactive slides are `aria-hidden` and their CTAs can't be reached with Tab. Only the first slide's headline is an `<h1>`; the other headlines are `<h2>`. Arrows and dots have accessible labels. Background images are decorative (`alt=""`).
12. AC12: With exactly one slide, arrows and dots are hidden and there is no autoplay.
13. AC13: If Strapi is unreachable or errors, or `homepage` has no slides, `/` still returns 200 and shows the existing "WESTLINE / Coming Soon" wordmark with waves as a fallback. The page never crashes.
14. AC14: The first slide's image is loaded with priority (no lazy loading) via `next/image`. The other slides lazy-load.
15. AC15: Content edits in Strapi show up on `/` within 60 seconds without a redeploy.

## Technical Approach

**Strapi (cms/)**. This is the first custom content type; Strapi 5.54.
- Component `home.hero-slide` (`cms/src/components/home/hero-slide.json`):
  - `headline`: string, required, max 80
  - `subtext`: text, optional, max 200
  - `image`: media, single, images only, required
  - `ctaLabel`: string, required, max 40
  - `ctaHref`: string, required (internal path like `/catalog` or absolute URL)
- Single type `homepage` (`cms/src/api/homepage/`):
  - `content-types/homepage/schema.json` with `kind: singleType` and `draftAndPublish: false`
  - Field `heroSlides`: repeatable `home.hero-slide`. Min 0, so the frontend fallback stays reachable.
  - Controller, route and service via `factories.createCore*`.
  - Named `homepage` (not `hero`) so later Home sections can add their own fields here.
- `cms/src/bootstrap/homepage.ts` is wired into `bootstrap()` in `cms/src/index.ts`, next to the existing `setup*` calls, and follows the idempotent pattern in `google-login.ts`. It does two things:
  1. Grants the `public` role `api::homepage.homepage.find` if it's missing, using `plugin::users-permissions.permission`.
  2. Seeds the three design slides only when the single type has no document. It uploads `cms/seed/home-hero/slide-{1,2,3}.jpg` through the upload plugin service, then calls `strapi.documents('api::homepage.homepage').create(...)`. The images are the three base64 backgrounds pulled out of the design HTML.
  - Seed CTA links: `/catalog`, `/surfboard-finder`, `/catalog`. These are routes that later specs will build. They 404 until then, which is accepted.
- API: `GET /api/homepage?populate[heroSlides][populate][image][fields][0]=url&…width,height`.

**Next.js (frontend/)**. Next 14 App Router, Tailwind 3.
- `frontend/src/lib/strapi/homepage.ts`:
  - `getHomepageHero(): Promise<HeroSlide[]>` is server-only. It fetches with `STRAPI_URL` (the same env var and default as `src/app/api/auth/*/route.ts`) using `next: { revalidate: 60 }` (AC15).
  - It normalizes the Strapi 5 flat response into `{ headline, subtext?, image: { src, width, height }, ctaLabel, ctaHref }` and makes relative `/uploads/...` URLs absolute against `STRAPI_URL`.
  - It returns `[]` and logs on network errors, non-2xx, 404 (no document) or malformed data, and drops slides with no image (AC13).
- `frontend/next.config.mjs` adds `images.remotePatterns` derived from `STRAPI_URL` (protocol, host, port, `/uploads/**`).
- `frontend/src/components/home/hero/`:
  - `hero.tsx` (server component): calls `getHomepageHero()` and renders `<HeroFallback/>` when there are no slides, otherwise `<HeroCarousel slides=…/>`.
  - `hero-fallback.tsx`: the existing Coming Soon wordmark and `Waves`, moved here from the catch-all page.
  - `hero-carousel.tsx` (`'use client'`): renders every slide in the markup for SSR (AC2), plus arrows, dots, the autoplay interval, hover/focus pause, swipe, `matchMedia` checks for reduced motion and hover, and the looping track with cloned first and last slides (the same approach as the design JS). Inactive slides get `aria-hidden` plus `inert` (AC11).
  - `hero-slide.tsx`: presentational. It renders `next/image` with `fill`, `sizes="100vw"`, `priority` on index 0, and object-cover, plus the scrim, the headline (`h1` at index 0, `h2` otherwise), optional subtext and a CTA. The CTA uses `next/link` for internal hrefs and a plain `<a>` for absolute URLs.
  - Motion: Tailwind transition utilities with arbitrary values (`duration-[11s]`, `scale-[1.08]`) and `motion-reduce:` variants. Any extra keyframes go in `tailwind.config.ts`.
  - Height: `min-h-[calc(100vh-64px)]`, because the existing SiteNav is a sticky white 64px bar. The design's transparent overlay nav is out of scope.
- `frontend/src/app/[locale]/[[...segments]]/page.tsx`: the Home branch renders `<Hero/>` in place of the inline Coming Soon markup. The page becomes `async`, and the `notFound()` guard is unchanged.

**Testing (TDD)**. Vitest + RTL + jsdom under `frontend/src/__tests__/`:
- Unit tests for the fetcher's normalization, URL rewriting and error paths, with `fetch` mocked.
- Component tests for the carousel: fake timers for autoplay, `matchMedia` stubs for reduced motion and hover, touch events for swipe.

## Out of Scope

- The rest of the Home page (Surfboard Finder teaser, trust marquee, Featured Gear), which will be separate specs.
- The site nav redesign: the design's transparent overlay nav and gradient over the hero, and the mega-menu. `site-nav.tsx` stays as is.
- The routes the CTAs point to (`/catalog`, `/surfboard-finder`), which the catalog-listing-page and surfboard-finder epics will build.
- The Product content model and the catalog itself.
- Per-slide focal point or background-position controls, multiple CTAs per slide, video slides.
- Localization of hero content (only the `en` locale exists).
- Draft/preview workflow for hero content (`draftAndPublish` is off).

## Standards Applied

- [global/git-workflow](../../../../../../standards/global/git-workflow.md): branch `feat/storefront-catalog/home-hero`, commit and PR conventions.

## Changelog

| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-09-27 | Ori Chai Matan | created | Initial shaping | — |

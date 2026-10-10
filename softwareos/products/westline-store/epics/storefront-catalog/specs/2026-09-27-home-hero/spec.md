# Home Hero

> Epic: ../../epic.md
> Status: in-progress
> Branch: feat/storefront-catalog/home-hero
> Covers: /, frontend/src/components/home/hero/**, frontend/src/lib/strapi/homepage.ts, cms/src/api/homepage/**, cms/src/components/shared/carousel-hero.json, cms/src/components/shared/button-cta.json, cms/src/components/shared/target.json, cms/src/bootstrap/homepage.ts, frontend/src/__tests__/components/home/hero/**, frontend/src/__tests__/lib/strapi/homepage.test.ts

## Overview

Replaces the Home page's "Coming Soon" placeholder with the design's full-bleed hero carousel. Each slide has a title, a background image and a primary CTA button. All slide content comes from a new Strapi single type `homepage` and is rendered server-side in Next.js. This is the storefront's entry point: its CTAs funnel shoppers into the catalog. It is also the first Strapi content type in the project, so it proves the Next.js → Strapi content pipeline (content type, public permission, fetch, `next/image`) end to end. It is **not** in the tech plan's Candidate Specs list; it was added as the epic's front door. It serves the success criterion ("a shopper can land on the catalog…") indirectly.

## Goals & User Stories

- As a shopper landing on `/`, I see a striking, on-brand hero that tells me what WESTLINE is and gives me one clear next step (shop, take the Finder, see a collection).
- As a shopper, I can move between hero slides with arrows, dots or a swipe, or just let them rotate.
- As a content editor, I can change hero titles, images and CTAs, add or remove slides, and reorder them in the Strapi admin without a code deploy.

## Acceptance Criteria

1. AC1: `/` renders a hero carousel whose slides (title, background image, CTA button text + link), in order, come from the Strapi `homepage` single type. No hero copy or image is hardcoded in the frontend.
2. AC2: The hero is server-rendered: every slide's headline and CTA are present in the initial HTML response, before JavaScript runs.
3. AC3: *(Removed 2026-09-28 — no seed. Hero content is entered manually in the Strapi admin; see Technical Approach.)*
4. AC4: The `homepage` content is publicly readable (`GET /api/homepage` works without auth) and read-only for the public role.
5. AC5: Visuals match the design's hero section. The hero fills the viewport below the site nav. Each slide shows a cover-fit background image, a left-to-right dark scrim, and a bottom-left white headline (Poppins display, `clamp(28px, 4.2vw, 56px)`). The primary button uses horizon blue with a horizon-deep hover, uppercase Inter 700. Arrows sit mid-left and mid-right (44px, 36px at ≤640px), and dots sit bottom-center.
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

**Strapi (cms/)**. This is the first custom content type; Strapi 5.54. The model was built in the Content-Type Builder and differs from the original shaping (see Changelog 2026-09-28).
- Component `shared.carousel-hero` (`cms/src/components/shared/carousel-hero.json`), one slide:
  - `Title`: string, required
  - `BackgroundImg`: media, single, required
  - `Button`: component `shared.button-cta`, single
- Component `shared.button-cta` (`cms/src/components/shared/button-cta.json`):
  - `Text`: string, required
  - `LinkUrl`: string (internal path like `/catalog` or absolute URL)
  - `targetLink`: component `shared.target` (enum `_blank` / `_self`), currently not read by the frontend
- Single type `homepage` (`cms/src/api/homepage/`):
  - `content-types/homepage/schema.json` with `kind: singleType` and `draftAndPublish: false`
  - Field `Hero`: repeatable `shared.carousel-hero`, required.
  - Controller, route and service via `factories.createCore*`.
  - Named `homepage` (not `hero`) so later Home sections can add their own fields here.
- `cms/src/bootstrap/homepage.ts` is wired into `bootstrap()` in `cms/src/index.ts`, next to the existing `setup*` calls, and follows the idempotent pattern in `google-login.ts`. It grants the `public` role `api::homepage.homepage.find` if it's missing, using `plugin::users-permissions.permission`.
- **No seed.** Content lives in the local Postgres DB, not in git. On a fresh DB, `/` shows the Coming Soon fallback (AC13) until an editor adds the slides in the admin: Content Manager → Homepage → add `Hero` entries (Title, BackgroundImg, Button text + link) using the three design slides ("Westline" → Shop Now → `/catalog`, "Find your Surfboard" → Take the Finder → `/surfboard-finder`, "The Dawn Patrol Collection" → Shop the Collection → `/catalog`). The CTA routes 404 until later specs build them, which is accepted.
- API: `GET /api/homepage?populate[Hero][populate][BackgroundImg]=true&populate[Hero][populate][Button]=true`.

**Next.js (frontend/)**. Next 14 App Router, Tailwind 3.
- `frontend/src/lib/strapi/homepage.ts`:
  - `getHomepageHero(): Promise<HeroSlide[]>` is server-only. It fetches through `strapiFetch` (`frontend/src/lib/strapi/client.ts`, owned by site-nav) with the default `revalidate: 60` (AC15) and 404 as a silent status.
  - It normalizes the Strapi 5 flat response from `Hero[]` (`Title`, `BackgroundImg`, `Button.Text`, `Button.LinkUrl`) into `{ headline, image: { src, width, height }, cta }` (`cta` via button-cta's `toButtonCta`) and makes relative `/uploads/...` URLs absolute through `strapiMediaUrl`.
  - It returns `[]` and logs on network errors, non-2xx, 404 (no document) or malformed data, and drops slides missing a title, image (with width/height), button text or link (AC13).
- `frontend/next.config.mjs` adds `images.remotePatterns` derived from `STRAPI_URL` (protocol, host, port, `/uploads/**`).
- `frontend/src/components/home/hero/`:
  - `Hero.tsx` (server component): calls `getHomepageHero()` and returns `<HeroFallback/>` when there are no slides, otherwise `<HeroCarousel slides=…/>`, in a single return.
  - `HeroFallback.tsx`: the existing Coming Soon wordmark and the waves, moved here from the catch-all page. The waves are a JSX constant (`WAVES`), not a component.
  - `HeroCarousel.tsx` (`'use client'`): renders every slide in the markup for SSR (AC2), plus arrows, dots, the autoplay interval, hover/focus pause, swipe, `matchMedia` checks for reduced motion and hover, and the looping track with cloned first and last slides (the same approach as the design JS). Inactive slides get `aria-hidden` plus `inert` (AC11). The arrow icons are `<Icon name="arrow-left" | "arrow-right" />` from `@/shared/components/icons`.
  - `HeroSlide.tsx`: presentational.
  - UI strings live in `hero-texts.ts` (`texts`): the carousel's `aria-label` "Featured" and `aria-roledescription` "carousel", "Previous slide", "Next slide", the "Slides" group label, `slide(n)` ("Slide N"), and the fallback's wordmark and message. `HeroCarousel` and `HeroFallback` import it. The slide headlines and CTA text are Strapi content, not texts. It renders `next/image` with `fill`, `sizes="100vw"`, `priority` on index 0, and object-cover, plus the scrim, the headline (`h1` at index 0, `h2` otherwise) and a CTA rendered by `<ButtonCTA>` (button-cta).
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
- [frontend/components](../../../../../../standards/frontend/components.md): one component and one return per file, PascalCase components / kebab-case TS, all Strapi fetches via `lib/strapi/client.ts`, all URLs via `lib/routes.ts`.

## Changelog

| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-09-27 | Ori Chai Matan | created | Initial shaping | — |
| 2026-09-28 | Ori Chai Matan | changed | Aligned spec with the built model: `homepage.Hero` (repeatable `shared.carousel-hero`: `Title`, `BackgroundImg`, `Button` → `shared.button-cta`: `Text`, `LinkUrl`) replaces `home.hero-slide`/`heroSlides`. Seed removed (AC3 dropped, `cms/seed/**` out of Covers); content entered manually in the admin. Subtext dropped (not in the design). Covers updated. | — |
| 2026-09-29 | Ori Chai Matan | change | Aligned with the frontend/components standard: renamed `hero.tsx`→`Hero.tsx`, `hero-slide.tsx`→`HeroSlide.tsx`, `hero-carousel.tsx`→`HeroCarousel.tsx`, `hero-fallback.tsx`→`HeroFallback.tsx` (tests `hero.test.tsx`→`Hero.test.tsx`, `hero-carousel.test.tsx`→`HeroCarousel.test.tsx`); `Hero` has a single return; `Waves` is a JSX constant; the arrow `ArrowIcon` is replaced by the shared `Icon`; `homepage.ts` fetches via `strapiFetch`. No behavior change — hero CTA markup byte-identical, all tests unchanged apart from import paths | frontend/components standard |
| 2026-09-29 | Ori Chai Matan | change | Moved the hero's hardcoded UI strings into `components/home/hero/hero-texts.ts` (`slide(n)` as a function); rendered text and aria attributes unchanged (live carousel HTML byte-identical); fallback message kept as the current "Something Wrong!" | frontend/components standard |
| 2026-09-29 | Ori Chai Matan | change | Fallback message aligned to AC13: hero-texts `fallbackMessage` 'Something Wrong!' → 'Coming Soon' — the code had drifted from the spec since 55de6d6 and the AC13 test was failing; corrects the 2026-09-29 texts row, which had kept the drifted text | — |
| 2026-10-06 | Ori Chai Matan | change | AC13 fallback background: `bg-[#F9F9F9]` → the page background token `bg-background` (test added). The wordmark and waves are unchanged | surfboard-detail-page |

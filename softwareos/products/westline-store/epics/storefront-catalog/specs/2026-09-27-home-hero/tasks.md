# Tasks — Home Hero

> Epic: storefront-catalog
> Spec: 2026-09-27-home-hero
> Branch: feat/storefront-catalog/home-hero

## Backend — Strapi homepage single type

- [x] T1 Add slide components — **changed 2026-09-28:** built as `shared.carousel-hero` (`Title`, `BackgroundImg`, `Button`) + `shared.button-cta` (`Text`, `LinkUrl`, `targetLink`) in the Content-Type Builder; `home/hero-slide.json` is unused and to be removed
- [x] T2 Add single type `cms/src/api/homepage/` (schema.json with `singleType`, `draftAndPublish: false`, repeatable `Hero` (was `heroSlides`); core controller/route/service factories)
- [-] T3 *(Dropped 2026-09-28 — no seed)* Extract the three hero background images from the design `index.html` base64 (`.bg-1/2/3`) into `cms/seed/home-hero/slide-{1,2,3}.jpg`
- [x] T4 Add `cms/src/bootstrap/homepage.ts`: idempotent public `find` permission grant (seed-if-empty dropped 2026-09-28); wire `setupHomepage(strapi)` into `cms/src/index.ts` bootstrap
- [x] T5 Manually verify `GET /api/homepage?populate…` against local Strapi (unauthenticated): 200, three slides, image url/width/height present; Record the response shape here. *(Note 2026-09-28: the verification below was against the original `heroSlides` model and seed; re-verify against `Hero` with manually entered content.)* **Verified 2026-09-27** against the running local Strapi/Postgres: unauthenticated `GET /api/homepage?populate[heroSlides][populate]=image` → 200 with `data: { id, documentId, createdAt, updatedAt, publishedAt, heroSlides: [{ id, headline, subtext: null, ctaLabel, ctaHref, image: { url: "/uploads/slide_N_<hash>.jpg", width, height, alternativeText: "", … } }] }`, 3 slides in design order; a reload kept the same `documentId` and 3 slides (no duplicate seed/uploads); public `PUT`/`DELETE /api/homepage` → 403 (AC4).

## Frontend — data layer

- [x] T6 Write failing tests `frontend/src/__tests__/lib/strapi/homepage.test.ts`: normalization, relative→absolute image URLs, missing subtext, slide without image dropped, `[]` on network error / non-2xx / 404 / malformed JSON, `revalidate: 60` passed to fetch
- [x] T7 Implement `frontend/src/lib/strapi/homepage.ts` (`getHomepageHero`, `HeroSlide` type) to pass T6
- [x] T8 Add `images.remotePatterns` for Strapi uploads (derived from `STRAPI_URL`) to `frontend/next.config.mjs`

## Frontend — hero UI

- [x] T9 Write failing tests `frontend/src/__tests__/components/home/hero/hero-carousel.test.tsx`: all slides rendered; h1 only on first; arrows + dots navigate and set `aria-current`; inactive slides `aria-hidden`/inert; autoplay 5s (fake timers); pause on focusin / hover-capable mouseenter; no autoplay under reduced motion; single slide hides controls; swipe ≥40px advances, <40px doesn't; internal CTA uses Link href
- [x] T10 Implement `hero-slide.tsx` + `hero-carousel.tsx` in `frontend/src/components/home/hero/` (Tailwind, looping cloned track, zoom/slide-in motion with `motion-reduce:` variants) to pass T9; match design visuals (AC5–AC6)
- [x] T11 Write failing tests `frontend/src/__tests__/components/home/hero/hero.test.tsx`: renders carousel when slides exist, renders Coming Soon fallback when `[]`
- [x] T12 Implement `hero.tsx` + `hero-fallback.tsx` (move `Waves` + Coming Soon markup out of the catch-all page) to pass T11
- [x] T13 Render `<Hero/>` in the Home branch of `frontend/src/app/[locale]/[[...segments]]/page.tsx` (async page, `notFound()` guard unchanged)

## Verification

- [~] T14 Run the full frontend test suite + coverage on changed files (testing skill); `npm run lint` and `npm run build` pass. 2026-09-27: 54/54 tests pass, lint clean (one pre-existing warning in `layout.tsx`), `tsc --noEmit` clean in frontend + cms; `next build` not yet run (it would clobber the running `next dev` `.next/`); coverage not yet collected
- [ ] T15 Run QA against acceptance criteria (qa skill), including manual browser checks for AC5/AC6/AC9/AC14/AC15 and the Strapi-down fallback (AC13)

## Tidy-up (frontend/components standard)

- [x] T16 Align with frontend/components: PascalCase renames (+ tests), single return in `Hero`, `WAVES` JSX constant, `ArrowIcon` → shared `Icon`, `homepage.ts` → `strapiFetch`. 2026-09-29: 124/125 (pre-existing AC13 failure only), tsc + lint clean
- [x] T17 Add `hero-texts.ts` and use it in HeroCarousel and HeroFallback. 2026-09-29: carousel suites green, live carousel HTML byte-identical; the pre-existing AC13 test (expects "Coming Soon") still fails as before

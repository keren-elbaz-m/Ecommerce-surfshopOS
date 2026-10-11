# Tasks — Button CTA

> Epic: storefront-catalog
> Spec: 2026-09-29-button-cta
> Branch: feat/storefront-catalog/button-cta

## Component

- [x] T1 Write failing tests `frontend/src/__tests__/shared/components/button-cta/button-cta.test.ts` (AC6: Text/LinkUrl → text/href; targetLink `_blank` → `_blank`, missing/null/`_self` → `_self`; null on missing Text, missing LinkUrl, missing button)
- [x] T2 Write failing tests `frontend/src/__tests__/shared/components/button-cta/ButtonCTA.test.tsx` (AC1–AC5: exact md class list; sm/lg padding + font only; internal `/`/`#` → next/link, `https://` → `<a>`; `_blank` → target + rel, `_self` → neither; tabIndex passthrough / absent by default)
- [x] T3 Implement `frontend/src/shared/components/button-cta/button-cta.ts` (types + `toButtonCta`) to pass T1
- [x] T4 Implement `frontend/src/shared/components/button-cta/ButtonCTA.tsx` + `index.ts` to pass T2 (the AC3 test mocks `next/link` with a marker so internal vs `<a>` is actually distinguished)

## Hero swap

- [x] T5 `frontend/src/lib/strapi/homepage.ts`: populate `Button.targetLink`, use `toButtonCta`, `HeroSlide.cta: ButtonCtaData` replacing `ctaLabel`/`ctaHref`; update `homepage.test.ts` for the new populate + shape, plus a new case: Strapi `targetLink: _blank` → `cta.target: '_blank'`
- [x] T6 `frontend/src/components/home/hero/hero-slide.tsx`: render `<ButtonCTA {...slide.cta} tabIndex={hidden ? -1 : undefined} />`, remove the inline classes/`isInternal`/Link branch; update `hero-carousel.test.tsx` + `hero.test.tsx` fixtures to `cta`
- [x] T7 Delete the empty untracked `frontend/src/shared/components/button/`

## Verification

- [x] T8 Run the full frontend suite (testing skill) — all new tests green, hero/homepage suites unchanged apart from fixtures (pre-existing hero AC13 failure noted, not fixed); `npm run lint`, `tsc --noEmit`. **2026-09-29:** 88/89 — the only failure is the pre-existing hero AC13 test; lint + `tsc` clean. AC8 checked live: the rendered hero CTA HTML (6 anchors = 4 slides + 2 clones) is byte-identical before vs after the swap
- [ ] T9 Run QA against acceptance criteria (qa skill), incl. a browser check that the hero buttons look identical and a `_blank` target opens a new tab (set one slide's targetLink in the admin, then revert)

## Tidy-up (frontend/components standard)

- [x] T10 Record home-hero's PascalCase renames in this spec; ButtonCTA already conforms to frontend/components. 2026-09-29: ButtonCTA tests green

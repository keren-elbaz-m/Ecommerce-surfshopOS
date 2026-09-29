# Button CTA

> Epic: ../../epic.md
> Status: in-progress
> Branch: feat/storefront-catalog/button-cta
> Covers: frontend/src/shared/components/button-cta/**, frontend/src/__tests__/shared/components/button-cta/**

## Overview

Extracts the home hero's call-to-action button into a reusable shared component, `ButtonCTA`, with a mapper from the existing Strapi component `shared.button-cta` (`Text`, `LinkUrl`, `targetLink`).

It wires up `targetLink`, which the hero has never populated or used, so an editor's "open in new tab" choice takes effect. The hero switches to `<ButtonCTA>` and must look and behave exactly as before.

It's not in the tech plan's Candidate Specs: it's a reuse refactor, so upcoming Home sections, catalog banners and the Finder teaser share one CMS-driven button. There's no Strapi schema change; `LinkUrl` stays optional.

## Goals & User Stories

- As a content editor, when I set a button's target to "_blank" in Strapi, the link opens in a new tab on the site.
- As a shopper, the hero buttons look and work exactly as before.
- As a developer building the next Home section or banner, I render `<ButtonCTA>` from a Strapi `shared.button-cta` with one mapper call. I don't copy the hero's classes and link logic.

## Acceptance Criteria

1. AC1: `ButtonCTA` (`frontend/src/shared/components/button-cta/ButtonCTA.tsx`, exported from `index.ts`) takes:
   - `text`
   - `href`
   - `target` (`'_self' | '_blank'`, default `'_self'`)
   - `size` (`'sm' | 'md' | 'lg'`, default `'md'`)
   - `tabIndex?`

   `size` is frontend-only, not a Strapi field.
2. AC2: The styling is fixed inside the component. At `size="md"` the class list is exactly the current hero button's: `inline-flex items-center gap-2 rounded-[9px] border-2 border-transparent bg-horizon px-[26px] py-[14px] text-sm font-bold uppercase tracking-[0.02em] text-horizon-ink no-underline hover:bg-horizon-deep`. `sm` and `lg` change only padding and font size.
   - **Note:** the `sm` and `lg` values are provisional. No design exists for them yet; only `md` comes from the design (the current hero button). Replace them when a design defines them.
3. AC3: Internal hrefs (starting with `/` or `#`) render through `next/link`. Any other href, such as external `http(s)://` URLs, renders as a plain `<a>`.
4. AC4: `target="_blank"` renders `target="_blank"` and `rel="noopener noreferrer"`. `_self` renders neither attribute.
5. AC5: `tabIndex` is passed through to the rendered link. When it's omitted, no `tabIndex` attribute is set.
6. AC6: `toButtonCta(button)` (`frontend/src/shared/components/button-cta/button-cta.ts`) maps a Strapi `shared.button-cta` to `{ text, href, target }`:
   - `text` ← `Text` and `href` ← `LinkUrl`.
   - `target` ← `targetLink.targetLink`: `'_blank'` gives `'_blank'`, and anything else (missing, `null`, `'_self'`) gives `'_self'`.
   - It returns `null` when `Text` or `LinkUrl` is missing or empty, or when the button itself is missing.
7. AC7: The home hero fetches `targetLink` (`populate[Hero][populate][Button][populate]=targetLink`). It renders every slide's CTA through `<ButtonCTA>` built from `toButtonCta`, and still skips any slide whose button maps to `null`.
8. AC8: No regressions in the hero:
   - The same markup, classes, text and hrefs.
   - Inactive and cloned slides still pass `tabIndex={-1}`.
   - The existing hero and homepage tests stay green, with fixtures updated only for the new `cta` shape.
   - A slide whose Strapi button has `targetLink: _blank` opens in a new tab.

## Technical Approach

**`frontend/src/shared/components/button-cta/`**
- `button-cta.ts` (pure TS, no JSX, safe to import from server libs):
  - `export type ButtonCtaTarget = '_self' | '_blank'`
  - `export interface ButtonCtaData { text: string; href: string; target: ButtonCtaTarget }`
  - `export interface StrapiButtonCta { Text?: string | null; LinkUrl?: string | null; targetLink?: { targetLink?: ButtonCtaTarget | null } | null }`
  - `export function toButtonCta(button?: StrapiButtonCta | null): ButtonCtaData | null` (AC6)
- `ButtonCTA.tsx` is a server-compatible component with no `'use client'`.
  - Props: `ButtonCtaData`-style `text` / `href` / `target?`, plus `size?` and `tabIndex?`.
  - Base classes are shared. Size classes:
    - `sm`: `px-[18px] py-[10px] text-xs`
    - `md`: `px-[26px] py-[14px] text-sm` (the current hero button)
    - `lg`: `px-[32px] py-[18px] text-base`
    - `sm` and `lg` are **provisional placeholders, not from a design**. A code comment next to the size map says so too.
  - `isInternal(href)`, moved from `hero-slide.tsx` (now `HeroSlide.tsx`), picks `next/link` or `<a>`. `target` and `rel` are applied on either element.
- `index.ts` exports `ButtonCTA`, `toButtonCta` and the types.

**Hero swap (files owned by home-hero, edited here)**
- `frontend/src/lib/strapi/homepage.ts`:
  - The query becomes `populate[Hero][populate][BackgroundImg]=true` plus `populate[Hero][populate][Button][populate]=targetLink`.
  - `StrapiHero.Button` becomes `StrapiButtonCta`.
  - `normalizeHero` uses `toButtonCta(Button)` and returns `null` if that returns `null`.
  - `HeroSlide` drops `ctaLabel` / `ctaHref` in favor of `cta: ButtonCtaData`.
- `frontend/src/components/home/hero/HeroSlide.tsx` (renamed from `hero-slide.tsx`): the inline `ctaClassName`, `isInternal` and `Link`/`<a>` branch are replaced by `<ButtonCTA {...slide.cta} tabIndex={hidden ? -1 : undefined} />`.
- The tests `homepage.test.ts` (populate assertion, expected objects), `HeroCarousel.test.tsx` and `Hero.test.tsx` (fixtures; renamed from `hero-carousel.test.tsx` / `hero.test.tsx`) switch to the `cta` shape. Their assertions otherwise stay the same.
  - The pre-existing `Hero.test.tsx` AC13 failure ("Coming Soon" vs "Something Wrong!") isn't this spec's concern and is left as is.

**Cleanup:** delete the empty, untracked `frontend/src/shared/components/button/` (`Button.tsx`, `index.ts`).

**Tests (TDD):**
- `frontend/src/__tests__/shared/components/button-cta/button-cta.test.ts` (node): AC6.
- `ButtonCTA.test.tsx` (jsdom + RTL): AC1–AC5, including an exact class-list assertion for `md` (AC2).

## Out of Scope

- Other variants (secondary, ghost, outline, ink), icons, and a loading state.
- Using ButtonCTA anywhere besides the hero (later sections adopt it in their own specs).
- Any Strapi schema change. `LinkUrl` stays optional, and `shared.target` is left as it is.
- Treating `mailto:` / `tel:` specially. They render as `<a>` like any non-internal href.

## Standards Applied

- [global/git-workflow](../../../../../../standards/global/git-workflow.md): branch `feat/storefront-catalog/button-cta`, commit and PR conventions.
- [frontend/components](../../../../../../standards/frontend/components.md): one component and one return per file, PascalCase components / kebab-case TS, all Strapi fetches via `lib/strapi/client.ts`, all URLs via `lib/routes.ts`.

## Changelog

| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-09-29 | Ori Chai Matan | created | Initial shaping | — |
| 2026-09-29 | Ori Chai Matan | change | Path references updated for home-hero's PascalCase renames (`hero-slide.tsx`→`HeroSlide.tsx`, `hero(-carousel).test.tsx`→`Hero(Carousel).test.tsx`); ButtonCTA itself already meets the frontend/components standard (one component, one return). No code or behavior change here | frontend/components standard |

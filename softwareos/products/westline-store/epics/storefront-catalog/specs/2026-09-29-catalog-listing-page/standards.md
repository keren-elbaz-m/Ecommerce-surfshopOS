# Standards for Catalog Listing Page

The following standards apply to this work.

---

## frontend/components

# Frontend Components

These conventions apply to the Next.js storefront (`frontend/src/`). They were codified on 2026-09-29 during the site-nav tidy-up, and home-hero, product-card, button-cta and site-nav follow them.

> **Pre-existing exception:** the auth area owned by account-signup-login (`app/[locale]/auth/**`, `app/api/auth/**`, `lib/auth/**`, `components/logout-button.tsx`) predates this standard. It will be aligned in a separate change. Don't restructure it as part of other work.

## One component, one return per file

A `.tsx` file defines **one** React component, and that component has **one** JSX `return`. An early guard `return null` is allowed.

Don't reach for a new component file to get there:

- **Lift per-item state to the parent.** When each item in a list needs state (open/closed, active, expanded), keep it in the parent as a single value or a keyed collection, and render the item inline.
  - `DesktopMenu` holds one open item (hovered / focused / dismissed hrefs) and **one** close timer, not a component per menu item.
  - `MobileMenu` keeps its expanded accordions in a `Set` of item hrefs, cleared when the panel closes.
- **Inline sub-markup in maps.** Render list items and columns directly inside `.map(...)` in the parent's single return, rather than in `Item` / `Column` components.
- **Use JSX constants for static markup.** Markup that takes no props is a module-level constant, not a component. Examples: `const WAVES = (…)` in `HeroFallback.tsx` and `const HAMBURGER = (…)` in `MobileMenu.tsx`.
- **Choose between variants in the single return.** Write `return cond ? <A/> : <B/>` rather than two returns (see `Hero.tsx` and `ButtonCTA.tsx`).
- **Use the single `Icon` component.** Icons are `<Icon name="…" />` from `@/shared/components/icons`. Its one internal map holds the SVG shapes (`heart`, `account`, `cart`, `arrow-left`, `arrow-right`).
  - To add an icon, add an entry to that map rather than writing an inline `<svg>` or an `XIcon` component.
  - Size, stroke and fill come from the caller's props. Icons are `aria-hidden`, and the surrounding control carries the accessible name.

## File naming

- **React components are PascalCase.** The file is named after its component: `Hero.tsx`, `HeroCarousel.tsx`, `ProductCard.tsx`, `SiteNav.tsx`.
- **Non-component TypeScript is kebab-case:** mappers, types, data and helpers such as `nav-menu.ts`, `product-card.ts`, `button-cta.ts`, `client.ts` and `routes.ts`.
- **Folders are kebab-case** (`site-nav/`, `product-card/`, `button-cta/`) and export through an `index.ts` barrel.
- **Tests mirror the source path** under `src/__tests__/` and keep the source file's name: `ProductCard.test.tsx`, `nav-menu.test.ts`.
- On macOS (a case-insensitive filesystem), do case-only renames in two steps so git records them: `git mv file.tsx file.tsx.tmp && git mv file.tsx.tmp File.tsx`.

## UI strings: `<name>-texts.ts`

UI strings live in a `<name>-texts.ts` next to the component. Interpolated strings are functions. CMS content never goes in texts files.

- Each component folder has one `<name>-texts.ts` that exports a `texts` object, and components import it instead of writing strings inline. This follows the auth area's existing `*-texts.ts` pattern. The files so far:
  - `site-nav/site-nav-texts.ts`, which is also used by `nav-menu.ts`
  - `home/hero/hero-texts.ts`
  - `product-card/product-card-texts.ts`
- **Every UI string goes there,** including visible text and all accessible names and descriptions (`aria-label`, `aria-roledescription`, visually hidden text).
- **Interpolated strings are functions,** for example `allCategory(name)`, `toggleSubmenu(item)`, `slide(n)`, `viewProduct(name)` and `showPhoto(n)`. Don't build them with inline template literals in components.
- **CMS content never goes in texts files.** Category, subcategory and product names, hero titles and button text all come from Strapi.
- A component that renders only props and CMS data, such as `ButtonCTA`, needs no texts file.
- Tests may import `texts` rather than repeating strings. Asserting the literal rendered string is also fine, and it's stricter.

## Where things live

| Path | Holds |
|---|---|
| `app/` | **Routing only**: pages, layouts and route handlers that compose components and call data functions. No reusable UI or business logic. |
| `components/<name>/` | App-level or layout components that appear once (site-nav, home/hero). |
| `shared/components/<name>/` | Reusable UI building blocks rendered in many places (product-card, button-cta, icons). |
| `lib/strapi/client.ts` | `STRAPI_URL` and `strapiFetch`: the **only** place that calls Strapi. |
| `lib/strapi/<resource>.ts` | One server-side data function per resource (`homepage.ts`, `navigation.ts`), built on `strapiFetch`. |
| `lib/routes.ts` | **Every** storefront URL: `productHref`, `categoryHref`, `subcategoryHref`, `genderHref`, `subcategoryGenderHref`, `GENDERS`, `GENDER_FILTER`, `GENDERED_CATEGORY_SLUGS`, and fixed links such as `CART_HREF`. |

A component's view type and its Strapi → view mapper live **in the component's folder** (for example `shared/components/product-card/product-card.ts`), not in a separate `features/` directory.

## Strapi fetches: `lib/strapi/client.ts`

- Every Strapi request goes through `strapiFetch(path, { label, fallback, revalidate?, silentStatuses? })`. Don't call `fetch` on Strapi, or read `process.env.STRAPI_URL`, anywhere else.
- `strapiFetch` never throws. On a network error, a non-2xx response or an unparsable body, it logs `[label] …` and returns `fallback`, so a page never breaks because Strapi is down. Pass expected statuses (such as `404` for an empty single type) in `silentStatuses`.
- `revalidate` defaults to 60 seconds.
- The body comes back unvalidated. The resource function checks its shape and normalizes it.
- The auth route handlers (`app/api/auth/**`) still call Strapi directly. That's part of the exception above.

## URLs: `lib/routes.ts`

- Don't write a storefront URL as a string literal in a component or mapper. Import a builder or constant from `@/lib/routes`.
- Reserved slug: the product slug `category` (it shares `/products/<slug>` with `/products/category/…`).
- Pages that serve these URLs (catalog-listing-page, product-detail-page) must match them.

---

## global/git-workflow

# Git Workflow

## Branch naming

One branch per spec, named from the SoftwareOS slugs (no date prefixes):

```
feat/<epic-slug>/<spec-slug>          # spec work — the canonical branch
feat/<epic-slug>/<spec-slug>--t<id>   # parallel work on one spec, split by task
hotfix/<epic-slug>/<bug-slug>         # bug fixes
chore/<slug>                          # everything else
```

- The canonical branch is recorded in the spec's `tasks.md` `> Branch:` line. To map a branch to a spec: match that line first; fall back to suffix-matching `<spec-slug>` against `feat/*/`.
- Slugs are load-bearing — never rename an epic or spec after branches exist.
- Never commit on the default branch. Branch first.

## Commits

Conventional-commit-ish titles, scoped to the epic:

```
feat(<epic-slug>): add password reset endpoint
fix(<epic-slug>): handle expired reset tokens
chore: bump CI node version
```

- Small, focused commits. Reference task IDs in the body when useful (`T3`, `T4.1`).
- **Never `git commit --no-verify`.** Failing hooks mean fix the cause, not skip the check.
- No force pushes to the default branch.

## Pull requests

- Open PRs with the **pr** skill — it runs verification, builds the title (`feat(<epic-slug>): <spec title>` / `fix(<epic-slug>): <bug title>`) and body from the spec, and writes the PR URL back to `tasks.md`.
- Tasks incomplete → open as draft.

## Production mode

When `softwareos/config.yml` has `mode: production`, the workflow tightens:

- **Git itself is never gated.** Commit and push as usual — pushing is how CI runs and how a branch gets reviewed, so blocking it would obstruct the verification the mode is asking for. The gates live in the tools that can actually check something.
- **A PR needs a passing `qa-report.md` and a recorded `> Reviewed:` line** (written by `/code-review`) before the pr skill will open it. A draft PR is for incomplete work, not for skipping verification.
- **Unfixed HIGH or CRITICAL review findings block the merge**, rather than being advice.

`/go-production` makes this transition and records any waivers; `/go-production --revert` goes back. Neither is a quiet config edit — both leave an entry in `softwareos/production-readiness.md`.

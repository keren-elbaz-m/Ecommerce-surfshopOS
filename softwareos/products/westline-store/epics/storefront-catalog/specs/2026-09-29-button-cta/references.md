# References for Button CTA

## Related Specs

- **[Related]** [2026-09-27-home-hero](../2026-09-27-home-hero/spec.md) owns the Strapi `cms/src/components/shared/button-cta.json` and `target.json` schemas, plus the hero files this spec edits (`hero-slide.tsx`, `lib/strapi/homepage.ts` and their tests). The two specs stay separate because this spec adds a shared component and only swaps the hero onto it. The schemas themselves don't change.

The Step 4 cross-tree coverage search found no formal matches: no spec claims `frontend/src/shared/components/button-cta/**`.

## Upstream Meeting

_None._

## Similar Implementations

### Current hero CTA

- **Location:** `frontend/src/components/home/hero/hero-slide.tsx`
- **Relevance:** it's the style and behavior source. It has the inline `ctaClassName`, the `isInternal` check (`/` or `#`) choosing between `next/link` and `<a>`, and `tabIndex={-1}` on hidden slides.

### Component + mapper folder

- **Location:** `frontend/src/shared/components/product-card/`
- **Relevance:** the folder pattern this spec follows: the component, its view type and its Strapi → view mapper live in one folder, and `index.ts` exports them.

### Hero normalizer

- **Location:** `frontend/src/lib/strapi/homepage.ts` (`normalizeHero`, `HOMEPAGE_QUERY`)
- **Relevance:** it gets the new `targetLink` populate and uses `toButtonCta`.

## Visuals

_None._ The source is the current hero button in code. Only `md` is from the design; `sm` and `lg` are provisional.

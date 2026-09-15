# Surfboard Finder — Tech Plan

> Epic: epic.md · Product: ../../mission.md

## Approach

A multi-step Next.js questionnaire collects answers (skill level, frequency, weight/height, conditions, budget); a matching function scores/filters catalog products by their Strapi attributes (from storefront-catalog's content model, e.g. volume/length/skill-level tags) and returns the top matches — no ML, just rule-based/weighted scoring.

## Architecture Impact

Likely needs to extend the Strapi Product content type (from storefront-catalog) with finder-relevant fields (e.g. recommended skill level, volume, board type) if not already present. Matching logic itself can live in Next.js — no new service.

## Risks & Unknowns

- The rule-based scoring/weighting needs real tuning against real product data to feel accurate rather than arbitrary — worth a quick spike with sample products and personas before committing to the exact scoring formula.
- Depends on storefront-catalog's Product content model already existing with the right attributes.

## Candidate Specs

- `board-finder-attributes` — extend the Strapi Product content type with finder-relevant fields (skill level, volume, length, board type, wave conditions) and seed sample data; no frontend/matching yet.
- `questionnaire-flow` — multi-step Next.js questionnaire UI that collects answers client-side, ending in a submit action; no matching/results yet.
- `board-matching-recommendations` — rule-based scoring function matching answers against real catalog boards, plus a results page with top recommendations; includes tuning the scoring weights against sample data.
- `recommendation-to-cart` — wires each recommendation to its product detail page and lets the shopper add it to cart directly from results.

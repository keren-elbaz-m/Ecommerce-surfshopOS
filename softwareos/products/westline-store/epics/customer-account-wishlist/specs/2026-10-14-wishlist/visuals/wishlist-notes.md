# Wishlist: design notes

Extracted from `westline-site-desing/` (all dated 28 Sep). These are the canonical sources for this spec.

| File | Used for |
|---|---|
| `catalog.html` | the card heart `.prod-wish` |
| `profile.html` | `#panelWishlist`, the slider, toast and remove scripts |
| `product-apparel-samurai-boardshort.html` | the standard PDP `.pdp-actions` / `.btn-wish` |
| `product-surfboard-tideline.html` | the surfboard PDP `.pdp-actions-row` / `.btn-outline-sm` |

> **Source note:** account-profile was shaped from the 20 Sep "My Account" artifact. That copy, and `storefront-catalog/.../home-hero/visuals/home.html`, show an older 34px borderless, white/90 heart. Every 28 Sep file uses the 36px bordered heart below, which is what the shared `ProductCard` already renders. There's no product-card gap.

## Card heart (`.prod-wish`): catalog, profile, PDP rails

- Position and size: `position:absolute; top:10px; right:10px; z-index:2`; 36×36, `border-radius:50%`.
- Surface: white, 1px `--border`, `box-shadow:0 1px 4px rgba(16,24,40,.10)`, `color:var(--horizon)`. Hover sets the border to horizon.
- SVG: 17px, `stroke:currentColor; fill:none; stroke-width:2`. `[aria-pressed="true"]` fills it with `currentColor`.
- Markup: `<button class="prod-wish" type="button" aria-label="Add to wishlist" aria-pressed="false">`. The script toggles `aria-pressed` and swaps the label between "Add to wishlist" and "Remove from wishlist".
- **Spec:** the shared `ProductCard` (product-card AC8) via `WishlistProductCard`. No change to the card.

## Account wishlist (`profile.html` `#panelWishlist`)

- **Head:** `h2.display` "Wishlist" + `span.count` "N saved". The script writes `n + ' saved'` for every n, including "1 saved".
- **Cards:** the shared card markup, with `aria-pressed="true"` and the label "Remove from wishlist" (`.js-wish-remove`). The photo dots are hidden when there's a single image.
- **Slider:** `.wish-slider-wrap{display:flex; align-items:center; gap:16px}`.
  - `.wish-grid{display:flex; gap:20px; overflow-x:auto; scroll-snap-type:x mandatory; padding-bottom:2px; flex:1 1 auto; min-width:0; scrollbar-width:none}` (with the webkit scrollbar hidden).
  - Cards: `flex:0 0 var(--wish-card-w, 240px); scroll-snap-align:start`.
- **Slider script:** `CARD_W 240`, `GAP 20`.
  - Desktop: the grid width is pinned to whole cards, `count = floor((available+GAP)/SLOT)`, `fitted = min(count*SLOT-GAP, available)`.
  - `(max-width:640px)`: one card at the full available width.
  - The arrows scroll by `grid.clientWidth`. `.is-disabled` (visibility hidden) applies at the start or end, or when nothing overflows. Layout reruns on resize and load.
- **Arrows:** `.slider-arrow` 36×36 round, white, 1px border, `box-shadow:0 2px 8px rgba(0,0,0,.1)`, 16px ink chevron (stroke 2). Labels: "Scroll wishlist left" / "Scroll wishlist right". Hidden at ≤640px.
- **Remove:** `.prod-card{transition:opacity .25s, transform .25s}`. `.is-removing` sets `opacity:0; transform:scale(.96)`, and after 250ms the card is removed and the count and empty state are re-checked.
- **Toast:** `.wish-toast`, fixed `bottom:24px`, centred, ink background, white 13.5px text, `padding:12px 22px`, radius 9px, `z-index:80`. It slides up from `translateY(20px)` with opacity `.2s` and stays 2.2s. Text: "Removed from wishlist."
- **Empty state:** `.wish-empty` (already built by account-profile): "Your wishlist is empty. Browse Surfboards or Apparel to add favorites."
- **Not built (out of scope):** `.js-wish-addcart` / the "Added to cart." toast (no add-to-cart from the wishlist).

## Standard PDP (`product-apparel-samurai-boardshort.html`)

- `.pdp-actions{display:flex; gap:12px; margin:22px 0 20px}` holds `<button class="btn-add-cart">Add to Cart</button>` (flex:1) and `<button class="btn-wish" aria-label="Add to wishlist">`.
- `.btn-wish{width:52px; border:1px solid var(--border); border-radius:var(--radius); background:var(--white)}`, centred. SVG 20px, `stroke:var(--ink); fill:none; stroke-width:1.8`. Hover sets the border to ink.
- **No saved state in the design** (no `aria-pressed`, no fill, no label swap).

## Surfboard PDP (`product-surfboard-tideline.html`)

- After `.price-buy-row` comes `.pdp-actions-row{display:flex; gap:10px; margin-top:16px}`, holding:
  - `<button class="btn-outline-sm" aria-label="Add to wishlist">` with a heart and the text "Wishlist"
  - `<button class="btn-outline-sm" aria-label="Share">` with a share icon and the text "Share" (**out of scope**)
- `.btn-outline-sm`:
  - Layout: `flex:1`, inline-flex centred, `gap:7px`.
  - Type: JetBrains Mono 11px/600, uppercase, `.05em`.
  - Surface: white, ink text, 1px border, radius 9px, `padding:11px 12px`. Hover sets the border to ink.
  - SVG: 15px, ink stroke 1.8, no fill.
- **No saved state in the design.** With Share removed, the single `flex:1` button fills the row.

## Gaps (not in any design; decided in shaping)

| Gap | Decision |
|---|---|
| PDP control saved state | Card convention: `aria-pressed="true"`, label "Remove from wishlist", heart filled with `currentColor`. Tideline keeps the "Wishlist" text |
| Order of saved items | Newest first |
| Save or remove failure | Revert quietly. A 401 behaves like a guest click |
| Sold-out / unpublished / deleted products | Unpublished or deleted drop out; sold-out stays a normal card |
| Toggle feedback on the catalog or PDP | None beyond the heart |
| Size limit | None |

## Also not built in the designs' pages

- Home "Featured gear" (`index.html`) and the PDP "You might also like" rails carry `.prod-wish` hearts. Those sections aren't built in the storefront, so there's nothing to wire. When they're built, they should use `WishlistProductCard`.

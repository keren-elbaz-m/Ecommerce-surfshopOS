# Standard PDP — design notes

Source: `westline-site-desing/product-apparel-samurai-boardshort.html`.

## Layout
- **Container:** `max-width:1240px`, `padding-inline:24px` (16px at ≤480).
- **Grid:** `.pdp-grid{grid-template-columns:1.6fr 1fr; gap:56px; align-items:start}`. At ≤900 it becomes `1fr`, gap 28px.
- **Buy column:** `position:sticky; top:88px`. Static at ≤900.
- **Breakpoints:** no 1100px breakpoint (unlike the surfboard page).

## Gallery
- **Desktop:** `grid-template-columns:1fr 1fr; gap:3px`. Items are `aspect-ratio:4/5`, `cursor:zoom-in`, buttons labelled "Photo n of 6". The design uses `background:cover`; we use a fixed 4:5 frame on `bg-background` with `object-contain`, because the seeded photos are 4:5.
- **≤900:** `display:flex; overflow-x:auto; scroll-snap-type:x mandatory`. Items `flex:0 0 100%; scroll-snap-align:center; scroll-snap-stop:always`. Scrollbar hidden. No dots, counter or arrows.
- **Lightbox:** same styles as the surfboard design. Opens at the clicked photo.

## Buy column
- **Eyebrow:** horizon mono 12px, `.14em`.
- **H1:** Poppins 800, uppercase, `clamp(26px,3.4vw,38px)`, line-height .95, margin `10px 0 12px`.
- **Price:** 24px/700, tabular-nums, margin `10px 0 16px`. No underline.
- **Size row:**
  - "Size — 36" label: mono 11px, uppercase, `.08em`, muted.
  - Grid: 5 columns (4 at ≤420), gap 8px.
  - Box: 1px border, radius 9px, mono 13px/600, padding `10px 4px`. Selected = ink background with white text.
  - Low stock: `.size-flag` (15px red circle with "!", top/right -6px) and `.low-stock-note` (mono 11.5px/600, uppercase, `.05em`, red).
  - The design has no sold-out style. Decided: disabled, muted, line-through.
- **Actions:** `.btn-add-cart` (flex:1) and a wishlist heart. Wishlist is hidden; Add to Cart is the shared ButtonCTA md at its natural width.
- **Trust list:** shared, with the surfboard copy ($75 / 30-day returns / 3–5 days). The boardshort design says $125 / final sale / 2–4 days.

## Accordions (`margin-top:32px`)
- **Header:** 15px/700, padding 20px 0, a 16px chevron that rotates 180°.
- **Group:** border-bottom.
- **Body:** muted 14.5px/1.7, `padding:0 0 22px`.
- Only one is open at a time.
- **Kept:** Description (open by default) and Shipping & Returns.
- **Dropped:** Details & Features (that content lives in the Description markdown).

## Hidden (decided in shaping)
- colour swatch (no Color field)
- Size Chart (no data)
- tag pill (Subtitle)
- wishlist
- "You might also like"
- the cart drawer

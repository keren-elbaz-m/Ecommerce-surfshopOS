# Cart — design notes

Sources (in `westline-site-desing/`, not copied here):
- `cart.html`: the cart page.
- `product-surfboard-tideline.html` (around lines 939–960) and `product-apparel-samurai-boardshort.html` (around lines 875–896): the cart drawer markup and JS.
- `promo-video/public/textures/live/pdp-cart-drawer-full.png`: a screenshot of the open drawer.

## Cart page
- **Grid:** `.cart-grid{grid-template-columns:1.7fr 1fr; gap:48px; align-items:start}`; one column at ≤900. H1 `clamp(28px,4vw,42px)`.
- **Free-shipping banner:** `--surface-2` box, 9px radius, `16px 18px` padding, 13px text; 5px bar on `--border` with a `--horizon` fill.
- **Rows:** `.cart-row{gap:20px; padding:24px 0; border-bottom}`. Photo 110×138 on `--surface-2`. Title 16px/700, meta 13px muted, price in JetBrains Mono 15px/600 (the line total; no unit price). Stepper: 32px buttons in a bordered 9px box. Remove: 12.5px muted underline.
- **Summary:** sticky `top:88px`, bordered, 24px padding, gap 16px. Rows: Subtotal / Shipping "Calculated at checkout" / Total (16px/700, border-top). Checkout: filled `--horizon`, full width. Continue Shopping: `.btn-cart-secondary`, 2px ink outline, inverts on hover. Note 12px muted.
- **Empty:** "Your cart is empty." plus Continue Shopping (max-width 220px); hides the list, banner and summary.

## Drawer
- **Panel:** fixed right, 420px (100% at ≤480), `translateX(100%)→0` over .25s, shadow `-8px 0 24px rgba(18,33,42,.12)`, z-index 61. Overlay `rgba(18,33,42,.5)`, z-index 60.
- **Content:** head "Your Cart (N)" with an X button; ship note 12.5px with a 4px bar; lines with a 76×95 photo, 14px title, 12.5px meta, 26px stepper, mono 13.5px line total, Remove below. Footer: Subtotal 14.5px/700, View Cart (filled), Checkout (outlined).
- **Behaviour:** opens on Add to Cart and from the header bag; closes on X, overlay or Esc; locks body scroll.

## Header badge
- `.cart-count`: JetBrains Mono 10px, `--horizon` circle 16×16, offset `left:-14px; top:-8px` from the bag; hidden at 0.

## Resolved in shaping / build
- **Free-shipping threshold:** $75, one constant. The design disagrees with itself: product-page text says $75, the drawer JS uses 100 and the cart JS uses 900.
- **Persistence:** the cart is persisted in Strapi; the design keeps it in memory.
- **Not built:** the tax line, promo code and recommended products aren't in the design and aren't built.
- **`--surface-2`:** became the `surface` Tailwind token (`#F9F9F9`).

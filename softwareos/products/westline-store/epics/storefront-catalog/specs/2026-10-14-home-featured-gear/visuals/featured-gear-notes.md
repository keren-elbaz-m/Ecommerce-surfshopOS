# Featured gear: design notes

Source: `westline-site-desing/index.html` (28 Sep), `section.featured`.

| Design | Spec |
|---|---|
| `section{padding-block:80px}`; `.featured{background:#F9F9F9}`; `.container` 1240px, 24px gutter | AC5 |
| `.section-head{max-width:640px; margin-bottom:40px}`; `.eyebrow` mono 12px/500 uppercase `.14em` horizon, margin-bottom 10px, "New in"; `h2.display` `clamp(28px,4.2vw,46px)` "Featured gear" | AC1 (copy from the Homepage, with a fallback) |
| `.prod-grid{display:grid; grid-template-columns:repeat(4,1fr); gap:20px}` | AC3 (4 per view; a slider at every width, Ori's decision) |
| `@media (max-width:980px)`: flex, gap 16px, `overflow-x:auto`, snap x mandatory, scrollbar hidden; card `flex:0 0 calc((100% - 16px)/2)`; `.prod-slider{padding-inline:52px}` (46px at ≤480px) | AC3 |
| `@media (max-width:520px)`: card `flex-basis:100%` | AC3 |
| `.prod-arrow`: absolute, `top:40%`, 40px round (36px at ≤480px), white, 1px border, `box-shadow:0 2px 10px rgba(16,24,40,.12)`, 18px ink chevron; prev left 0 / next right 0; hidden at either end. Labels "Previous products" / "Next products" | AC3 |
| `.prod-dots`: `margin-top:20px`, one `.slider-dot` per snap position (`Math.round(max / step) + 1`), "Go to slide N", `aria-current`; hidden when < 2 | AC3 |
| Cards: the shared `.prod-card` with `.prod-wish` | AC2 (`WishlistProductCard`) |

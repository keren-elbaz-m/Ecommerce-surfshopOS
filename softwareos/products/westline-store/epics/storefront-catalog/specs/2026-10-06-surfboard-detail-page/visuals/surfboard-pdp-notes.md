# Surfboard PDP: design notes

Source: `westline-site-desing/product-surfboard-tideline.html` (a sibling folder of the repos). These notes are extracted from it for the parts this spec builds. Site header, footer, "You might also like" and the cart drawer are excluded.

## Tokens (`:root`)

- **Colours:**
  - `--ink #101828`, `--muted #667085`, `--border #D0D5DD`, `--white #FFFFFF`
  - `--surface-2 #F9F9F9` (also the body background)
  - `--horizon #155EEF`, `--horizon-deep #0E40A3`, `--horizon-ink #F5F9FA`
- **Shape:** `--radius 9px`, `--maxw 1240px`.
- **Fonts:** body is Inter, `.display` is Poppins 800 uppercase, and mono is JetBrains Mono.
- **`.eyebrow`:** mono, 12px, weight 500, uppercase, `letter-spacing:.14em`.
- **`.container`:** max-width 1240px, padding-inline 24px (16px at ≤480px).

## Breadcrumb

- **Structure:** `Home / <Category> / <Product>`. Links are muted with no underline and turn horizon on hover. `.sep` is at opacity .5, and `.current` is ink.
- **Type:** mono, 12px, uppercase, `.04em`, flex with an 8px gap, `padding-block: 20px`.

## Grid: `.pdp` > `.container.pdp-grid3`

- **`.pdp`:** `padding-top: 8px; padding-bottom: 64px`.
- **Columns:** `330px 1fr 380px`, gap 44px, `align-items: start`. DOM order is desc, image, buy.
- **≤1100px:** `280px 1fr 340px`, gap 30px.
- **≤900px:** a single column with gap 32px. The order becomes image (1), buy (2), desc (3).
- **Sticky:** the image and buy columns are `position: sticky; top: 96px` above 900px and static below.

## Description column

- **Category eyebrow:** coloured horizon.
- **H1 `.display`:** `clamp(24px,2.1vw,30px)`, margin-top 8px.
- **`.pdp-desc-rule`:** 64×2px ink, opacity .9, margin-top 16px.
- **`.video-block`:**
  - Container: 16:9, radius 9px, `#0A1519` background, margin-top 48px (24px at ≤900px).
  - Video: `<video controls playsinline preload="metadata">`, `object-fit: cover`. No poster, no autoplay.
- **`.desc-copy`:**
  - "From the shaper" eyebrow in horizon, margin-bottom 10px.
  - The paragraph is 14px, line-height 1.7, muted, clamped to 6 lines (`-webkit-line-clamp: 6`).
- **`.desc-toggle`:**
  - Type: mono, 11.5px, weight 600, uppercase, horizon.
  - Icon: a 13px chevron (the shared `arrow-right` path) that rotates 90° when open.
  - Text: "Read more" / "Read less".
  - Accessibility: the design has no `aria-expanded`. The spec adds it.

## Attributes (`#attributesSection`)

- **Wrapper:** margin-top 48px, padding-top 36px, border-top. The "Attributes" eyebrow is horizon, margin-bottom 26px.
- **`.attr-section`:** padding 44px 0 with a bottom border (the first has no top padding; the last has no border).
- **Section titles:** h3 `.display`, `clamp(26px,3.2vw,34px)`, margin-bottom 28px.
- **Items:** a column with a 36px gap.
- **`.fit-label`:** mono, 11.5px, weight 700, uppercase, `.07em`, ink.
- **`.fit-track`:** 9px tall, `--border` colour, radius 5px, margin `16px 0 10px`.
- **`.fit-fill`:** horizon, full height, radius 5px, `width: <value>%`.
- **`.fit-marker`:**
  - Size: 19px circle, horizon, 3px white border.
  - Shadow: `0 0 0 1.5px horizon, 0 2px 5px rgba(18,33,42,.22)`.
  - Position: `left: <value>%; top: 50%; transform: translate(-50%,-50%)`.
- **`.fit-scale`:** mono, 10.5px, muted, uppercase, `.03em`.

| Section | Label | Data | Scale labels |
|---|---|---|---|
| Wave | Size | WaveSize | Knee / Double+ |
| Wave | Break | Break | Point / Reef / Beachbreak |
| Wave | Power | Power | Weak / Mushy · Medium / Steep · Strong / Barrels |
| Performance | Approach | Approach | Vertical / Pocket · Power / Carving · Cruisy / Positional |
| Performance | Skill Level | SkillLevel (enum) | Beginner / Intermediate / Advanced (design: Novice / Intermediate / Expert) |
| Performance | Foot Orientation | FootOrientation | Back Foot / Neutral / Front Foot |
| Shape | Foil / Rails | Foil | Thin / Medium / Full |
| Shape | Nose Shape | NoseShape | Pointed / Hybrid / Round |
| Shape | Tail Width | TailWidth | Narrow / Medium / Wide |
| Shape | Entry Rocker | EntryRocker | Relaxed / Medium / Aggressive |
| Shape | Exit Rocker | ExitRocker | Relaxed / Medium / Aggressive |
| Shape | Rocker Style | RockerStyle | Staged / Continuous |

**Deliberate deviations:**
- **Scale label positions.** The design lays scale labels out as a 3-column text grid (`1fr 1fr 1fr`, or `1fr 1fr` for 2 labels). The build places them at their true points instead: start at 0%, middle centred at 50%, end right-aligned at 100%. The fill and marker use the exact value, with no rounding or edge insetting (spec AC10).
- **Fin setup.** The design's `.fin-setup` block (diagram, "Thruster · 3-fin" and blurb) is **not built** (spec AC12).

## Gallery (`.pdp-image-col`)

- **Layout:** flex, centred, gap 14px (8px at ≤900px). It is prev button → main image → next button.
- **`.img-nav`:** a 36px circle, 1px border, white background. Hover sets the border to ink. Icon: a 16px chevron.
- **`.gallery-main`:**
  - Container: radius 9px, `#F9F9F9` background, `cursor: zoom-in`.
  - Image: `object-fit: contain; max-height: 74vh`.
- **Behaviour:**
  - No dots, thumbnails or counter on the page.
  - Prev/next wrap around.
  - Clicking the image opens the lightbox.

## Buy panel (`.buy-panel`)

- **Panel:** 1px border, radius 9px, padding `22px 22px 24px`, transparent background.
- **`.variant-label`:** mono, 11px, uppercase, `.08em`, muted, margin-bottom 8px.
- **`.variant-select`:**
  - Inter 14.5px, weight 600, 1px border, radius 9px.
  - Padding `12px 36px 12px 14px`, a chevron at the right, `appearance: none`.
- **Options:** formatted `6'0" · 29.4L`. Sold-out handling is a spec decision; the design has none.
- **`.price-buy-row`:** border-top, padding-top 18px, flex `space-between`, gap 14px, wraps.
- **`.pdp-price`:** 24px, weight 700, tabular-nums, `border-bottom: 2px solid ink`.
- **`.btn-add-cart`:**
  - Inter 700 13.5px, uppercase, `.03em`, horizon background, horizon-ink text.
  - Radius 9px, padding `14px 26px`. Hover sets horizon-deep.
- **`.pdp-trust`:**
  - Layout: border-top, padding-top and margin-top 18px, a column with an 8px gap.
  - Items: 13px muted, each with a 15px horizon icon.
  - Text: "Free shipping over $75", "30-day returns", "Ships in 3–5 business days".
- **Not built:** the Material and Fin System selects, "More options ↓", Wishlist and Share.

## Lightbox (`#lightbox`)

- **Container:** `role="dialog"`, `aria-modal="true"`, fixed, inset 0, z-index 200, `rgba(8,16,20,.94)` background.
- **Stage:** `min(88vw,860px) × min(78vh,860px)`. The photo uses `object-fit: contain` on `#F9F9F9` with radius 9px.
- **Close button:** top -52px, right 0, 36×36, white X.
- **Arrows:** 46px circles, `rgba(255,255,255,.12)` background (.22 on hover), `1px solid rgba(255,255,255,.35)` border. They sit at ±64px outside the stage, and 8px inside at ≤900px.
- **Counter:** "n / N" at bottom -34px, mono, 12px, white.
- **Behaviour:**
  - It closes on the close button, a backdrop click or Escape.
  - ArrowLeft and ArrowRight navigate and wrap, and stay in sync with the main image.
  - Body `overflow: hidden` while open.
  - The design has no focus handling. The spec adds focus to the close button on open and back to the main image on close.
- **Photos only:** no video.

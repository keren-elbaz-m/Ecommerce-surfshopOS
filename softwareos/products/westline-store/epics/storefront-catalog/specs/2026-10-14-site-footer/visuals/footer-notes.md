# Footer: design notes

Source: `westline-site-desing/index.html` (28 Sep), `footer`.

| Design | Spec |
|---|---|
| `footer{background:#fff; border-top:1px solid var(--border); padding-block:56px 28px}` | AC1, AC6 |
| `.foot-grid{grid-template-columns:1.4fr 1fr 1fr 1fr; gap:32px; margin-bottom:40px}`; `@media (max-width:760px){grid-template-columns:1fr 1fr}` | AC6 |
| `.foot-brand .word` Poppins 800 22px "WESTLINE"; `p` muted 14px, max 32ch, margin-top 10px: "Find Your Line. Surf equipment and lifestyle gear for the progressing surfer." | AC1 (tagline from Strapi) |
| `.foot-col h4` JetBrains Mono 11px uppercase `.08em` muted, margin-bottom 14px | AC1 (12px on the design system scale) |
| `.foot-col ul` flex column gap 10px; `a` 14px ink, hover horizon | AC1, AC4 |
| Columns: Shop (New Arrivals, Surfboards, Wetsuits, Apparel), Help (Surfboard Finder, Shipping & Returns, Size Guide, Contact), Account (Sign In, Create Account, Wishlist, Order History) | AC3: only links to existing pages are seeded |
| `.foot-bottom` border-top, padding-top 22px, 12.5px muted, flex space-between wrap gap 12px: "© 2026 WESTLINE. Find Your Line." + `.legal` (Privacy, Terms, Seller Login), gap 18px | AC1, AC3 (12px; no legal links seeded) |
| profile.html uses a minimal footer (copyright + Seller Login only) | replaced by the full footer everywhere (AC7) |

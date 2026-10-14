# My Account: design notes

Source: the claude.ai artifact **"My Account — WESTLINE"** (https://claude.ai/artifact/E3r2qw8MexsU3sXUgvBWCL, 20 Sep). The full HTML (7.3 MB, images inlined) isn't kept in the repo. The newer `westline-site-desing/profile.html` (28 Sep) has the same structure; the wishlist spec treats it as canonical.

Tokens: ink `#101828`, muted `#667085`, border `#D0D5DD`, horizon `#155EEF` / deep `#0E40A3` / ink `#F5F9FA`, success `#3F7D4F`, radius 9px, max width 1240px with a 24px gutter (16px at ≤480px).

| Design | Spec |
|---|---|
| `.breadcrumb`: JetBrains Mono 12px uppercase `.04em`, muted, `/` separators at 50% opacity, current item in ink, padding 20px | AC2, shared `ProductBreadcrumb` |
| `.acct-hero`: flex `space-between` `align-items:flex-end`, wraps; `padding-bottom:28px`, border-bottom, `margin-bottom:32px`. `h1.display` Poppins 800 uppercase `clamp(28px,4vw,40px)`; `.lede` 14.5px muted, `margin-top:8px` — "Welcome back, Jamie Rivera." | AC2 |
| `.btn-signout`: JetBrains Mono 12px uppercase `.06em`, transparent, 1px border, `padding:11px 18px`, ink border on hover; 14px spinner; script sets "Signing out…" and disables it | AC3 |
| `.acct-tabs`: flex, border-bottom, `margin-bottom:36px`, `overflow-x:auto`. `.tab-btn` 13.5px/700 uppercase `.04em`, muted, `padding:12px 4px`, `margin-right:28px`, 2px bottom border; active = ink text + horizon border. "My Account" (overview), "Account Details" (details) | AC4 |
| Hash script `resolveHash`: wishlist/orders → overview + `scrollIntoView` after 50ms; details/account → details; else overview. Tab click → `history.replaceState('#'+tab)`; listens to `hashchange` | AC5 |
| `.acct-form` max-width 560px; `.auth-row` flex gap 14px (stacks at ≤480px); `.auth-field` label 12.5px/600 muted, input 14.5px, `padding:12px 14px`, focus ring `0 0 0 3px rgba(21,94,239,.12)`. Fields: First name, Last name, Email, **Phone (omitted: no user field)** | AC6 |
| `.btn-auth`: horizon, 14.5px/700 uppercase `.03em`, `padding:15px 26px`, `margin-top:8px`, 16px spinner; "Save Changes" → "Saving…"; `.acct-note` 12.5px/600 success, "Profile updated.", hides after 2.5s | AC8 |
| `#panelWishlist` (`order:1`): "Wishlist" `h2.display` `clamp(20px,2.6vw,26px)` + `.count` mono 12px "N saved"; `.wish-empty` dashed border, padding 32px, 14.5px muted, centred, max-width 560px: "Your wishlist is empty. Browse **Surfboards** or **Apparel** to add favorites." | AC13 (data: wishlist spec) |
| `#panelOrders` (`order:2`, `margin-top:56px`): "Order History" head + order accordion + demo note. **No empty state in the design** (gap: "You haven't placed any orders yet.") | AC13 (data: order-history spec) |
| `footer` border-top, `margin-top:40px`; `.foot-bottom` flex `space-between`, `padding:22px 0`, 12.5px muted: "© 2026 WESTLINE. Find Your Line." + "Seller Login" (**out of scope**) | AC14 |
| Nav `.js-nav-account` / `.js-nav-wishlist` active-icon toggling | out of scope (site-nav) |

**Gaps filled minimally:** the Order History empty state, and the save-error copy ("Couldn't save your changes. Please try again.").

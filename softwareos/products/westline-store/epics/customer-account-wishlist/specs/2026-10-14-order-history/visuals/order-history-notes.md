# Order History: design notes

Source: `westline-site-desing/profile.html` (28 Sep), `#panelOrders` (`order:2`, `margin-top:56px`).

| Design | Spec |
|---|---|
| `.acct-section-head` "Order History" (`h2.display`) | AC1 (heading moves from account-profile) |
| `.order-list{display:flex; flex-direction:column; gap:14px; max-width:760px}` | AC1 |
| `.order-card{border:1px solid var(--border); border-radius:9px; background:#fff; overflow:hidden}`; `.is-open` rotates the chevron | AC2, AC4 |
| `.order-summary` `<button aria-expanded aria-controls>`: `padding:16px 20px; display:flex; align-items:center; gap:16px; flex-wrap:wrap; text-align:left` | AC2, AC4 |
| `.order-preview` 52×52, radius 9px, `#F9F9F9`, `img{object-fit:cover}` | AC2 (first line's image) |
| `.order-main{flex:1 1 140px}`: `.order-no` JetBrains Mono 700 14.5px ("WL-104233"); `.order-date` 12.5px muted, `margin-top:3px` ("Placed Aug 22, 2026") | AC2 |
| `.order-mid` 13px muted ("3 items" = Qty 2 + Qty 1; "1 item"); `@media (max-width:600px){display:none}` | AC2 |
| `.order-status` Mono 11px uppercase `.06em`, `padding:6px 12px`, radius 20px: `.delivered` `rgba(63,125,79,.12)` / `#3F7D4F`; `.shipped` `rgba(21,94,239,.12)` / horizon; `.processing` `#F9F9F9` / muted | AC3 (cancelled = processing style, **gap**) |
| `.order-right{margin-left:auto; gap:16px}`: `.order-total` 700 15px; `.order-chevron` 16px muted stroke 2, `transition:transform .2s`, 180° when open | AC2, AC4 |
| `.order-detail{max-height:0; overflow:hidden; transition:max-height .25s ease}`; script sets `max-height = scrollHeight` | AC4 |
| `.order-detail-inner{padding:4px 20px 22px; border-top:1px solid var(--border)}` | AC5 |
| `.order-items` gap 14px, `margin-top:18px`; `.order-item` flex gap 14px; `.oi-thumb` 44px radius 9px surface (img cover) or `.placeholder` Mono 11px 700 muted initials ("RG", "WAX"); `.oi-name` 13.5px 600; `.oi-spec` Mono 11.5px muted ("Men's · Size M · Qty 1"); `.oi-price` `margin-left:auto` 700 13.5px | AC5 (spec = "{size} · Qty N"; initials rule **gap**: up to two initials) |
| `.order-totals{max-width:280px; margin-left:auto; margin-top:18px; padding-top:16px; border-top; gap:8px}`; rows 13px muted; `.total` ink 700 14.5px + border-top. "Subtotal", "Shipping" ("Free"), "Total" | AC5 |
| `.order-detail-note` "Track shipment" → toast "Live tracking isn't available in this demo." | **omitted** (V1 exclusion) |
| `<p class="acct-note show" style="color:var(--muted); margin-top:20px; max-width:560px">Order status shown here is for demo purposes and does not reflect live carrier tracking.</p>` | AC6 (only with ≥1 order) |
| Empty state | not in the design: account-profile's "You haven't placed any orders yet." (`.wish-empty` style) |

**Gaps:** Cancelled pill (grey like Processing); the initials rule; date format (`en-US`, short month, UTC); the item count is the sum of quantities.

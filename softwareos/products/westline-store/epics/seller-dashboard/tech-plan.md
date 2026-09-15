# Seller Dashboard — Tech Plan

> Epic: epic.md · Product: ../../mission.md

## Approach

Use Strapi's built-in admin panel (with a seller/admin role via Users & Permissions) as the seller dashboard itself — sellers log into Strapi admin directly to manage Products, inventory fields, and Orders, rather than building a custom Next.js dashboard UI.

## Architecture Impact

Configures Strapi admin roles/permissions to scope a seller role to Product, inventory, and Order management; adds an order status field/workflow to the Order content type (from cart-checkout) if not already present. No new services or infra.

## Risks & Unknowns

- Strapi's default admin panel may be clunky for non-technical sellers to manage inventory efficiently at scale — worth validating it's good enough for a 4-week capstone demo rather than over-engineering a custom UI.
- Need to make sure inventory decrements/stays consistent with orders placed through checkout.

## Candidate Specs

- `seller-role-access` — create a seller/admin role in Strapi Users & Permissions scoped to Product + Order content types, seed/login a seller admin account; no dashboard capability yet, just gated access.
- `product-catalog-management` — grant the seller role full CRUD on Products (including inventory/stock fields) through Strapi admin.
- `order-status-workflow` — add an order status field/enum (e.g. pending → fulfilled → shipped) to the Order content type, exposed as an editable field in Strapi admin.
- `order-fulfillment-view` — configure the Strapi admin Order list/detail views (filters, visible fields) so a seller can see incoming orders and update their status.
- `inventory-order-consistency-check` — verify/wire inventory decrements from checkout stay consistent with seller-driven stock edits (no negative stock, no silent overwrite races) — a small hardening pass validated via demo scenario.

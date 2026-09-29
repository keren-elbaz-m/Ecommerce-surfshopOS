# Cart & Checkout — Tech Plan

> Epic: epic.md · Product: ../../mission.md

## Approach

Cart state moves from purely client-side (storefront-catalog) to persisted (e.g. localStorage or a server-side cart tied to a session); checkout submits the cart to a new Strapi Order content type with a mock payment step (no real gateway); order confirmation reads the created order back from Strapi.

## Architecture Impact

Adds a new Order (and related line-item) content type/schema in Strapi to record checkouts. No new services or infra — still Next.js + Strapi + PostgreSQL.

## Risks & Unknowns

- Deciding where cart state lives (localStorage vs server-side) and how it survives page reloads is worth a quick spike.
- Without customer accounts yet, checkout needs a way to identify the guest order (e.g. guest checkout with email).
- Inventory isn't yet tracked against orders (Seller Dashboard epic).

## Candidate Specs

- `persistent-cart` — move cart state from client-only to persisted (localStorage/session-backed), with a viewable cart page for adjusting quantities and removing items.
- `order-content-model-admin-view` — Order + line-item content type/schema in Strapi with validation, plus a basic admin-panel view so orders are inspectable.
- `guest-checkout-flow` — checkout form (shipping/contact info, guest email) and mock payment step that submits the cart as a new Order in Strapi.
- `order-confirmation` — confirmation page that reads the created order back from Strapi and displays the summary.

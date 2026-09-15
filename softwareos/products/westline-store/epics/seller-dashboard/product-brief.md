# Seller Dashboard — Product Brief

> Epic: epic.md · Product: ../../mission.md

## Goals

So far, products and orders only exist as data seeded/created directly in Strapi — there's no way for a seller to manage the catalog or view/fulfill incoming orders through the product itself. Goal: give a seller a login-gated dashboard to manage products/inventory and view/update orders.

## User Stories

- As a seller, I can log in to a dashboard so I have a dedicated management area.
- As a seller, I can create/edit/delete products and update inventory so the catalog stays accurate.
- As a seller, I can view incoming orders and update their status (e.g. fulfilled/shipped) so I can fulfill purchases.

## Success Criteria

- A seller can log in, create/edit a product (reflected live in the storefront catalog), and see a real order placed via cart-checkout, updating its status — all persisted in Strapi.

## Out of Scope

- Custom-built Next.js seller UI
- Multi-seller / marketplace support
- Analytics/reporting dashboards
- Shipping label generation

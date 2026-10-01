# Customer Account & Wishlist — Tech Plan

> Epic: epic.md · Product: ../../mission.md

## Approach

Use Strapi's built-in Users & Permissions plugin for signup/login/session (JWT), add a Wishlist relation/content type tied to the user, and link existing guest orders (from cart-checkout) to the logged-in user's order history where possible.

## Architecture Impact

Enables/configures Strapi's Users & Permissions plugin (auth, JWT) and adds a new Wishlist content type/relation tied to Users and Products. No new services or infra — still Next.js + Strapi + PostgreSQL.

## Risks & Unknowns

- JWT storage/session handling in Next.js (cookies vs localStorage) needs a decision.
- Linking pre-existing guest orders from cart-checkout to a newly created account is not guaranteed to be possible (no account existed at order time) — scoped as best-effort, not guaranteed.

## Candidate Specs

- `account-signup-login` — enable Strapi Users & Permissions (JWT) and build Next.js signup/login/logout with session handling.
- `account-profile-management` — authenticated profile page to view/edit account details (name, email, address book).
- `wishlist-save-view-remove` — new Wishlist content type/relation (User↔Product); save/view/remove wishlist items.
- `order-history` — authenticated order history page reading the user's past orders from Strapi, including best-effort linking of pre-account guest orders.

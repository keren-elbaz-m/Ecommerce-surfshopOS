# Dependencies

## Internal

(none) — `web` and `cms` don't call each other yet; first checkout item for any spec that needs them wired.

## External

| Service | Purpose | Auth method | Docs |
|---|---|---|---|
| Mock payment provider | Simulated checkout/payment flow for V1 | TBD — not yet added to `cms/.env.example` | TBD |

`@strapi/plugin-users-permissions` is installed and gives JWT-based auth/roles out of the box — the natural fit for customer-account-wishlist rather than adding a separate auth library.

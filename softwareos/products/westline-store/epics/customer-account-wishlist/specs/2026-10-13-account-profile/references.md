# References for Account Profile

## Related Specs

- **[Related]** [cart-checkout / 2026-10-11-checkout](../../../cart-checkout/specs/2026-10-11-checkout/spec.md): owns `frontend/src/lib/strapi/account.ts`. This spec adds `updateMe` there and reuses `getMe` and the checkout page's auth-guard pattern. Kept separate because profile editing belongs to this epic, and the edit is one added function.
- **[Related]** [storefront-catalog / 2026-09-29-site-nav](../../../storefront-catalog/specs/2026-09-29-site-nav/spec.md): owns `frontend/src/lib/routes.ts`. This spec adds `ACCOUNT_HREF` and `ACCOUNT_PROFILE_API_HREF`. The nav's `WISHLIST_HREF` already targets `/account#wishlist`, which this spec makes real. Kept separate because these are additive constants only.
- **[Related]** [2026-09-22-account-signup-login](../2026-09-22-account-signup-login/spec.md): reuses `/api/auth/logout`, `getSession`, `loginHref` and `?next=` without modifying them.

## Upstream Meeting

_None._

## Similar Implementations

### Checkout page auth guard

- **Location:** `frontend/src/app/[locale]/checkout/page.tsx`
- **Relevance:** the same server-side gate and prefill.
- **Key patterns:** `getSession()` → `getMe(jwt)` → `redirect(loginHref(HREF))`, `dynamic = 'force-dynamic'`, noindex metadata.

### Checkout route handler + shared validation

- **Location:** `frontend/src/app/api/checkout/route.ts`, `frontend/src/lib/checkout/validate.ts` (+ `validate-texts.ts`)
- **Relevance:** the same cookie → validate → Strapi → mapped-error flow.
- **Key patterns:** reading the JWT from `SESSION_COOKIE_NAME`, `{ error: { code, fields } }` bodies, NO_STORE headers, and one validator used by both the form and the route.

### Cart / order custom Strapi API

- **Location:** `cms/src/api/cart/` (`identity.ts`, `routes/cart.ts`)
- **Relevance:** custom `auth: false` routes whose controller resolves the shopper from the Bearer JWT itself.
- **Key patterns:** `createIdentify(strapi)` to get `{ kind: 'user', userId, … }`; no core CRUD router, so nothing is exposed by id.

### FormField

- **Location:** `frontend/src/shared/components/form-field/FormField.tsx`
- **Relevance:** a labelled input with an inline error and `aria-invalid`, matching the design's `.auth-field`.

## Visuals

- `visuals/my-account-notes.md`: design → spec mapping from the "My Account — WESTLINE" claude.ai artifact (the 7.3 MB HTML isn't kept) (https://claude.ai/artifact/E3r2qw8MexsU3sXUgvBWCL), the design source. In scope from it: the breadcrumb, `.acct-hero` with Sign Out (spinner, "Signing out…"), `.acct-tabs` with `resolveHash` (details/account → details; wishlist/orders → overview + scroll), the `.acct-form` "Save Changes"/"Saving…"/"Profile updated." (auto-hides after 2.5s), the overview section heads (Wishlist above Order History via `order`), the `.wish-empty` copy and the `.foot-bottom` bar.
  - Omitted or deferred: the Phone field (no user field), the wishlist cards and slider, the order accordion and demo note (later specs), the nav icon active state (site-nav) and the footer "Seller Login" (seller-dashboard).
  - Design gaps filled minimally: the Order History empty state and the save-error copy.

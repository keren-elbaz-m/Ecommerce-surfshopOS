# Account Profile

> Epic: ../../epic.md
> Status: in-progress
> Branch: feat/customer-account-wishlist/account-profile
> Covers: /account, frontend/src/app/[locale]/account/**, frontend/src/app/api/account/**, frontend/src/components/account/**, frontend/src/lib/account/**, frontend/src/__tests__/components/account/**, frontend/src/__tests__/lib/account/**, frontend/src/__tests__/app/api/account/**, frontend/src/__tests__/app/[locale]/account/**, cms/src/api/account/**, cms/tests/account/**

## Overview

The authenticated **My Account** page at `/account`. It has a hero with a welcome line and Sign Out, two hash-routed tabs (**My Account** overview and **Account Details**), and a details form that saves first name, last name and email to the shopper's Strapi user. The overview tab renders the Wishlist and Order History sections with their headings and empty states only. Their data comes from the wishlist and order-history specs, which fill these sections in. The epic's tech-plan candidate is `account-profile-management`. The address book it mentions isn't in the design and isn't built here.

Design source: the "My Account — WESTLINE" artifact (`visuals/my-account-notes.md`). Every element traces to it, with these deliberate deviations:
- **Phone is omitted.** The design shows a Phone field, but the Strapi user model has none, and adding one was declined while shaping.
- **Design gaps, filled minimally:** the Order History empty-state copy and the save-error copy (the design has neither).
- The nav-icon active state and the footer "Seller Login" link from the design are out of scope.

Serves the epic success criterion "a shopper can … edit their profile … with all data persisted in Strapi and tied to their account."

## Goals & User Stories

- As a signed-in shopper, I can open My Account and see that it's my account ("Welcome back, {first} {last}.").
- As a signed-in shopper, I can update my first name, last name and email so my details stay current.
- As a signed-in shopper, I can sign out from My Account.
- As a shopper following a `/account#wishlist` or `/account#orders` link (nav, emails, later specs), I land on the right tab and section.

## Acceptance Criteria

1. AC1: Visiting `/account` without a valid session redirects server-side to `/auth/login?next=%2Faccount`. After signing in, the shopper returns to `/account`.
2. AC2: The page shows a breadcrumb "Home / My Account" (Home links to `/`) and a hero with the heading "My Account" and "Welcome back, {firstName} {lastName}." from the signed-in user.
3. AC3: Clicking **Sign Out** shows a spinner and "Signing out…" and disables the button. It calls the existing logout route, which clears the session, then lands on Home in a signed-out state.
4. AC4: There are two tabs, "My Account" and "Account Details". Exactly one tab and its panel are active, the active tab is underlined in horizon blue, and clicking a tab switches panels and replaces the URL hash (`#overview` / `#details`) without adding a history entry.
5. AC5: Hash routing, on load and on `hashchange`: `#details` or `#account` → Account Details; `#wishlist` or `#orders` → My Account, smooth-scrolled to that section; any other hash or none → My Account.
6. AC6: The Account Details form shows First name, Last name and Email, prefilled from the signed-in user. There is no Phone field.
7. AC7: On submit, client validation rejects an empty first or last name, a name over 50 characters, an empty email or an invalid email format. Errors show inline under each field and no request is sent.
8. AC8: While saving, the button shows a spinner and "Saving…" and is disabled. On success it reverts to "Save Changes", "Profile updated." appears for about 2.5 seconds, and the hero welcome line shows the new name.
9. AC9: The save persists to the shopper's own Strapi user (`firstName`, `lastName`, `email`, with `username` kept equal to `email`). Reloading the page shows the saved values.
10. AC10: The server re-validates every field (same rules as AC7) and ignores any field other than firstName, lastName and email. A shopper can only change their own account, never another user's by id.
11. AC11: Changing the email to one already used by another account shows the inline error "An account with this email already exists." under Email and saves nothing.
12. AC12: Any other save failure (Strapi down, an unexpected error) shows "Couldn't save your changes. Please try again." in the note slot, styled as an error, and keeps the entered values. An expired session redirects to sign-in.
13. AC13: The My Account tab shows **Wishlist** first ("Wishlist" heading plus a "0 saved" count) with the design's empty state ("Your wishlist is empty. Browse Surfboards or Apparel to add favorites." with links to those catalog pages). **Order History** follows, with the heading and the empty state "You haven't placed any orders yet." (design gap, minimal copy).
14. AC14: A minimal footer bar shows "© 2026 WESTLINE. Find Your Line."
15. AC15: The layout matches the design at desktop, 900px and 375px. At ≤480px the first/last name row stacks and the side gutter is 16px. The tabs scroll horizontally rather than wrap, and nothing scrolls horizontally at page level.

## Technical Approach

**Backend (`cms`) — new `cms/src/api/account/`** (no content type, custom routes only, like `api/cart`):
- `routes/account.ts`: `PATCH /account/profile`, `config: { auth: false }`. The controller resolves the user itself, as cart and order do.
- `controllers/account.ts`: resolves the shopper from the Bearer JWT. Reuse `createIdentify` from `api/cart/identity.ts`, rejecting anything that isn't `kind: 'user'` with 401. Read only `firstName`, `lastName` and `email` from the body, validate (`account-logic.ts`), lowercase and trim the email, check uniqueness against `plugin::users-permissions.user` excluding the user (409 `email-taken`), then update `{ firstName, lastName, email, username: email }` by the user's id. Return `{ firstName, lastName, email }`.
- Why not Strapi's built-in `PUT /api/users/:id`: it would need the Authenticated role's `user.update` permission, which has no ownership check, so any shopper could edit any user. The custom route only ever writes `ctx`'s own user.
- The admin-panel lockdown in `cms/src/extensions/users-permissions/strapi-server.ts` only wraps the admin Content Manager controller, so it's unaffected and stays untouched.

**Frontend (`frontend/src`):**
- `lib/account/validate-profile.ts` + `validate-texts.ts`: shared client/server validation (required, max 50, email format), mirroring `lib/checkout/validate.ts`. It returns `{ ok, value } | { ok: false, fields }`.
- `lib/strapi/account.ts`: add `updateMe(jwt, profile)` built on `strapiRequest('account/profile', { method: 'PATCH', … })`, mapping 200, 401, 409 `email-taken` and other statuses to a typed result. `getMe` is reused as it is. This is an edit in the checkout spec's file.
- `app/api/account/profile/route.ts`: `PATCH`. It reads `SESSION_COOKIE_NAME` (401 if absent), validates the body (400 with `fields`), calls `updateMe`, and maps the results: 409 → `fields.email`, 401 → 401, other errors → 502. Responses are no-store. Same shape as `app/api/checkout/route.ts`.
- `lib/routes.ts`: add `ACCOUNT_HREF = '/account'`, `ACCOUNT_PROFILE_API_HREF = '/api/account/profile'` and `LOGOUT_API_HREF = '/api/auth/logout'` (for Sign Out), and point `WISHLIST_HREF` at `` `${ACCOUNT_HREF}#wishlist` ``. This is an edit in the site-nav spec's file.
- `app/[locale]/account/page.tsx`: routing only, following `app/[locale]/checkout/page.tsx`. It uses `getSession()` → `getMe(jwt)`, does `redirect(loginHref(ACCOUNT_HREF))` when there's no account, sets `dynamic = 'force-dynamic'` and noindex metadata "My Account — WESTLINE", and renders `<AccountPage account={…} />`.
- `components/account/` (app-level, used once; follows the components standard: one component and one return per file, PascalCase, texts file, barrel):
  - `AccountPage.tsx` (client): breadcrumb, hero, tabs and both panels, plus the footer bar. It owns the active tab via `use-account-tab.ts` and the displayed name (updated on save).
  - `use-account-tab.ts`: the hash routing (`resolveHash`, initial read, `hashchange` listener, `history.replaceState` on tab click, `scrollIntoView` for wishlist/orders).
  - `SignOutButton.tsx`: POSTs to `/api/auth/logout`, then `router.push(HOME_HREF)` + `router.refresh()`. It doesn't reuse `components/logout-button.tsx`: that is the nav's control in the auth-area exception, with different markup and copy.
  - `use-save-profile.ts`: the form's values, errors, status and the save round trip (mirrors checkout's `use-place-order.ts`).
  - `AccountDetailsForm.tsx`: uses `FormField` from `shared/components/form-field` for the three inputs, a submit state machine (`idle | saving | saved | error`) and focus on the first invalid field.
  - `AccountOverview.tsx`: the Wishlist and Order History section heads and empty states, as static structure the later specs replace.
  - `account-texts.ts`: every string, including the interpolations `welcome(first, last)` and `savedCount(n)`.
  - `index.ts` barrel.
- Styling: Tailwind with the existing tokens (ink, horizon, muted, border, success, `rounded-[9px]`), matching the design classes (`.acct-hero`, `.tab-btn`, `.btn-signout`, `.btn-auth`, `.acct-note`, `.wish-empty`, `.foot-bottom`). Fonts: Poppins (display), Inter (body) and JetBrains Mono (breadcrumb, count, sign out), as already loaded.
- After a successful save, `router.refresh()` so server-rendered state matches the new values.

## Out of Scope

- Wishlist data, cards, slider and add/remove: the favorites/wishlist spec.
- Order data, the accordion, the order detail and the "demo purposes" order note: the order-history/checkout orders spec.
- Live shipment tracking: a V1 exclusion.
- A Phone field and an address book.
- The nav account/wishlist icon active state per tab (owned by storefront-catalog/site-nav).
- The footer "Seller Login" link (seller-dashboard epic, no seller login yet).
- Password change (handled by the existing forgot/reset flow) and email re-verification on change (email confirmation was removed in account-signup-login).

## Standards Applied

- [frontend/components](../../../../../../standards/frontend/components.md): one component per file, texts files, `app/` as routing only, Strapi via `lib/strapi/client.ts`, URLs via `lib/routes.ts`.
- [global/git-workflow](../../../../../../standards/global/git-workflow.md): branch `feat/customer-account-wishlist/account-profile`, commit and PR conventions.
- [global/tech-stack](../../../../../../standards/global/tech-stack.md): Next.js + Strapi + PostgreSQL, no new dependencies.

## Changelog

| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-10-13 | Ori Chai Matan | created | Initial shaping | — |
| 2026-10-13 | Ori Chai Matan | change | Implemented T1-T21 on `feat/customer-account-wishlist/account-profile`, stacked on `feat/cart-checkout/checkout` (main lacks the catalog/cart/checkout work this builds on). Status → in-progress. Added `use-save-profile.ts`, `LOGOUT_API_HREF`, the `cms/tests/account/**` and `__tests__/app/[locale]/account/**` tests to the plan/Covers. Greeting falls back to "Welcome back." for a nameless (Google) account. T22 (formal QA) open | tasks.md |
| 2026-10-14 | Ori Chai Matan | change | Design-source note: this spec was shaped from the 20 Sep "My Account" artifact, which predates `westline-site-desing/profile.html` (28 Sep), the canonical source used by 2026-10-14-wishlist. The only difference found is the card heart (34px borderless vs 36px bordered), which this spec does not render; nothing here changes | 2026-10-14-wishlist |
| 2026-10-14 | Ori Chai Matan | change | Merged hotfix `30-day-sessions` into this branch (`079b33c`). T22 QA: PASS WITH NOTES, 15/15 AC | qa-report.md |
| 2026-10-14 | Ori Chai Matan | change | wishlist: `AccountOverview` now renders `WishlistSection` (owned by the wishlist spec) for its wishlist half, taking a `wishlist` prop that `AccountPage` passes down from `app/[locale]/account/page.tsx` (`getWishlist(jwt)`, empty on failure). The static "0 saved" head/empty state and their strings/styles moved to `components/wishlist`; AC13's copy and structure are unchanged. Tests updated (`AccountOverview`, `AccountPage`, page +1) | customer-account-wishlist/2026-10-14-wishlist |

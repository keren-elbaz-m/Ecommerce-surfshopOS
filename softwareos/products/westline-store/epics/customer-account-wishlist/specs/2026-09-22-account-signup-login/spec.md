# Account Signup & Login

> Epic: ../../epic.md
> Status: in-progress
> Branch: feat/customer-account-wishlist/account-signup-login
> Covers: /auth/login, /auth/register, /auth/forgot-password, /auth/reset-password, /auth/connect/google/redirect, frontend/src/app/[locale]/auth/**, frontend/src/lib/auth/**, frontend/src/app/api/auth/**, frontend/src/components/logout-button.tsx, frontend/src/app/[locale]/layout.tsx, cms/config/plugins.ts, cms/config/middlewares.ts, cms/config/server.ts, cms/src/extensions/users-permissions/**, cms/src/index.ts, cms/src/bootstrap/welcome-email.ts

## Overview

Enables Strapi's built-in Users & Permissions plugin (JWT) for shopper accounts and builds the Next.js signup/login/logout flow on top of it, including a reusable server-side "require auth" helper that the epic's later specs (profile management, wishlist, order history) will use to protect their own pages. This is the first spec in customer-account-wishlist — everything else in the epic depends on a logged-in user existing.

Note: an empty branch `feat/customer-account-wishlist/login-signup` already exists (no commits beyond the initial scaffold) — it predates this spec and doesn't match the canonical branch name below (`account-signup-login` vs `login-signup`). Recommend deleting/renaming it to the canonical name before starting T1 of tasks.md, per the git-workflow standard ("slugs are load-bearing").

## Goals & User Stories

- As a shopper, I can sign up with first name, last name, email, and password so I have a persistent identity.
- As a shopper, I can log in with my email and password so I can access my account.
- As a shopper, I can log out so my session ends.
- As a developer building the rest of this epic, I have a reusable "require auth" check so profile, wishlist, and order-history pages can protect themselves without re-deciding session handling.

## Acceptance Criteria

1. AC1: A visitor can create an account at `/signup` with first name, last name, email, and password (min 8 characters); on success they are logged in and redirected to the homepage.
2. AC2: Signup rejects a duplicate email with a clear inline error.
3. AC3: Signup rejects a password under 8 characters with a clear inline error before the request reaches Strapi.
4. AC4: A registered shopper can log in at `/login` with email + password; on success they are redirected to the homepage.
5. AC5: An invalid email/password combination shows a clear inline error and does not create a session.
6. AC6: On successful signup or login, the session is stored as an httpOnly, secure cookie set by a Next.js route handler — the JWT is never exposed to client-side JS.
7. AC7: A logged-in shopper can log out, which clears the session cookie and returns them to a logged-out state.
8. AC8: The nav's account icon reflects logged-in vs. logged-out state.
9. AC9: A reusable server-side "require auth" helper exists that redirects unauthenticated requests to `/login`, ready for future authenticated pages to use.
10. AC10: The "Forgot your password?" link on `/login` navigates to `/forgot-password` (real flow, no longer a stub).
11. AC11: A shopper can request a reset at `/forgot-password` by email; the UI always shows a generic "check your email" confirmation regardless of whether the email exists (anti-enumeration, matching Strapi's own `/api/auth/forgot-password` behavior).
12. AC12: When a matching account exists, Strapi generates a reset token and sends a real email (via Resend) containing the reset link to the account's email address.
13. AC13: Visiting `/reset-password?code=<token>` with a valid, unexpired code shows a "set new password" form (new password + confirm, min 8 characters).
14. AC14: A mismatched password/confirmation, or a password under 8 characters, shows a clear inline error before the request reaches Strapi.
15. AC15: Submitting a valid new password with a valid code updates the password, logs the shopper in (same httpOnly-cookie pattern as login/register), and shows a success state.
16. AC16: Visiting `/reset-password` with a missing code shows a clear "link is invalid or expired" state rather than attempting the request.
17. AC17: An invalid/expired/already-used code shows a clear inline error and does not change the password or create a session.
18. AC18: A shopper's password cannot be changed via the Strapi Admin panel's Content Manager — any request to the admin User-edit endpoint containing a password value is rejected server-side, regardless of the admin's role (including Super Admin).
19. AC19: Both `/login` and `/signup` show a "Continue with Google" button below the existing form.
20. AC20: Clicking it navigates the browser to Strapi's `/api/connect/google` to begin the OAuth flow.
21. AC21: On successful Google sign-in, the browser lands on `/connect/google/redirect`, which exchanges Strapi's session-backed OAuth result for our own httpOnly session cookie (same pattern as login/register/reset-password) and redirects to `/`.
22. AC22: If Google sign-in fails or is cancelled, `/connect/google/redirect` shows a clear error state with a link back to `/login`, rather than a broken page.
23. AC23: Without `GOOGLE_CLIENT_ID`/`GOOGLE_CLIENT_SECRET` configured, the button still renders but clicking it fails at Strapi with "This provider is disabled" — not a crash. The feature self-activates once real credentials are added to `cms/.env`; no further code changes needed.
24. AC24: When a new account is created — via password signup (`/api/auth/local/register`) or first-time Google OAuth signup — a welcome email is sent to that account's email address.
25. AC25: The welcome email includes a "Shop Now" button linking to the home page, and is personalized with the shopper's first name when available (password signups only — Google's default profile mapping never includes it — falling back to a generic greeting otherwise).
26. AC26: The welcome email uses the same design tokens and table-based inline-styled HTML as the reset-password email.
27. AC27: A failure to send the welcome email never breaks or blocks signup itself — both the password and Google OAuth paths still succeed even if the email send fails.

## Technical Approach

**Backend (`cms`):**
- Use `@strapi/plugin-users-permissions`'s built-in `/api/auth/local/register` and `/api/auth/local` endpoints for signup/login — no custom content-type needed for accounts.
- `cms/config/plugins.ts` already sets `jwtManagement: 'refresh'` and `sessions.httpOnly: true` for `users-permissions` — verify/extend this config to support the refresh-token flow this spec needs, rather than reconfiguring from scratch.
- Configure `cms/config/middlewares.ts`'s `strapi::cors` to allow the `web` origin with credentials — required for cross-origin cookies between `web` and `cms` in local dev.

**Frontend (`web`):**
- Since `web` and `cms` are separate origins, Strapi can't set an httpOnly cookie directly on `web`. Add Next.js Route Handlers under `web/src/app/api/auth/{register,login,logout}/route.ts` that proxy to Strapi's REST endpoints server-side, then set the JWT as an httpOnly, secure, `sameSite` cookie on `web`'s own response.
- Pages: `web/src/app/(auth)/login/page.tsx` and `web/src/app/(auth)/signup/page.tsx`, matching the provided mockups (login: email + password + stubbed "forgot password"; signup: first/last name + email + password with an 8-char hint, no confirm-password or terms checkbox).
- `web/src/lib/auth/session.ts` — server-side helper to read/validate the session cookie, used both for gating pages and for the nav's logged-in/out state.
- Add `STRAPI_URL` (server-only env var) to `web` for the route handlers to call `cms`.
- Update the nav's account icon to reflect session state (mirrors the mockups' `is-active-icon` treatment).

**Forgot/reset password (`cms` + `web`):**
- Strapi's built-in `/api/auth/forgot-password` (`{ email }`, always returns `{ ok: true }`, anti-enumeration by design) and `/api/auth/reset-password` (`{ password, passwordConfirmation, code }`, returns `{ jwt, user }` — reissues a session since `jwtManagement: 'refresh'`) — no custom content-type needed.
- **Email delivery:** `cms/config/plugins.ts` configures `provider: 'nodemailer'` (`@strapi/provider-email-nodemailer`, the official Strapi package) against Resend's documented SMTP relay (`host: smtp.resend.com`, `port: 465`, `secure: true`, username the literal string `"resend"`, password `RESEND_API_KEY`) — real emails are actually sent. `defaultFrom` is `onboarding@resend.dev`, Resend's own sandbox sender that needs no domain verification but can only deliver to the Resend account's own signup email until a real domain is verified (see Out of Scope). Chose the official nodemailer provider over three low-adoption third-party Resend packages (checked npm download counts and dependency freshness before picking).
- `cms/providers/console-log/` (the previous provider) stays installed but unused — switching back is a one-line change to `plugins.ts`'s `provider` value.
- `cms/src/index.ts`'s `bootstrap()` sets the Users-Permissions plugin's `advanced.email_reset_password` store setting to `${CLIENT_URL}/reset-password` on boot (idempotent), so the logged reset link points at the real frontend page instead of Strapi's placeholder default.
- Route handlers `web/src/app/api/auth/{forgot-password,reset-password}/route.ts`, mirroring the register/login proxy pattern; reset-password's success response sets the httpOnly session cookie exactly like login/register.
- Pages `web/src/app/(auth)/{forgot-password,reset-password}/page.tsx`, matching the provided mockups' in-place confirmation-state swap. `reset-password` reads `code` from the URL query string; adapted from the mockup to handle a missing/invalid code (the mockup assumed a working static demo flow) and drops the mockup's "Open Reset Link" button (which assumed no real backend) since we don't expose the token client-side — the confirmation copy instead points at the `cms` server console.

**Admin panel password lockdown (`cms`):**
- Strapi's Content Manager uses a dedicated controller for editing Users-Permissions users (`content-manager-user.js`, registered as `contentmanageruser`) that accepts a `password` field in its update body, gated only by the admin's RBAC field permissions — which Super Admin always bypasses by design, so RBAC alone can't fully close this off for a solo project where the one admin account is Super Admin.
- Chosen mechanism: `cms/src/extensions/users-permissions/strapi-server.ts` wraps that controller's `update` action to reject (400, with an explicit message pointing at the self-service flow) any request whose body has a non-empty `password` (mirroring Strapi's own null/empty no-op convention), before delegating to the original controller. This is enforced in code, independent of role/RBAC configuration, so it also covers Super Admin. Note: must be `.ts`, not `.js` — this project's build only compiles TS sources into `dist/`, and Strapi's extension loader specifically globs for a compiled `strapi-server.js` there; a hand-written `.js` file placed directly under `src/extensions/` never reaches `dist/` and would be silently never loaded (caught during verification, not assumed).
- Documented alongside it, as a one-time manual admin-setup recommendation (not automated, since Super Admin's permissions aren't configurable): also uncheck the `password` field's "Update" permission for any additional non-Super-Admin roles via Settings → Administration Panel → Roles — defense-in-depth for whenever this project has more than one admin.

**Google OAuth login (`cms` + `web`):**
- Mechanism, confirmed by reading Strapi's oauth-connect source directly (not assumed): `GET {STRAPI_URL}/api/connect/google` (full-page redirect, not fetch) starts the flow; Google redirects to `{STRAPI_URL}/api/connect/google/callback` (Strapi's own callback, handled entirely server-side — this exact URL must be registered as the Authorized redirect URI in Google Cloud Console); on success Strapi stores the OAuth result in its own session and redirects the browser, with no tokens in the URL, to the "front-end callback URL" configured per-provider (set to `${CLIENT_URL}/connect/google/redirect`). That `web` page then does a credentialed `GET` (`credentials: 'include'`, so Strapi's session cookie rides along) to `{STRAPI_URL}/api/auth/google/callback`, which returns `{ jwt, user }` — finally POSTed to a new `web/src/app/api/auth/oauth/complete/route.ts` that sets our own httpOnly cookie exactly like every other auth route in this spec.
- `cms/src/index.ts`'s `bootstrap()` additionally configures the Users-Permissions plugin's grant store for `'google'` (`enabled`, `key`, `secret`, `callback = ${CLIENT_URL}/connect/google/redirect`) — but only when `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET` are present in the environment; otherwise it logs a warning and leaves the provider disabled (Strapi's own built-in behavior), so the app never breaks on missing credentials — which is exactly the case right now.
- New public env var `web/NEXT_PUBLIC_STRAPI_URL` (Strapi's origin, exposed to the browser by design — it's not a secret) — needed because the "Continue with Google" link and the redirect page's fetch both run client-side, unlike every other route in this spec which proxies server-side through `STRAPI_URL`.
- Accepted, considered default: Strapi's reset-password flow doesn't check `provider`, so a Google-only shopper can use `/forgot-password` to add a local password afterward, letting them subsequently also log in with email+password. This is Strapi's own default behavior, costs no extra code, and isn't harmful — documented rather than restricted.
- Not affected: the AC18 admin-panel lockdown already blocks any password write regardless of a user's provider, Google-authenticated included.

**Welcome email (`cms`):**
- Mechanism: `strapi.db.lifecycles.subscribe({ models: ['plugin::users-permissions.user'], afterCreate })` registered in `cms/src/index.ts`'s `bootstrap()`. This is a DB-level hook — it fires on any creation of a `up_users` row regardless of which controller/service triggered it, which is exactly why one hook covers both `/api/auth/local/register` and Google OAuth's first-time `providers.connect()` user creation with no duplicated per-route code.
- Extracted to a new file, `cms/src/welcome-email.ts` (registered from bootstrap) — unlike the smaller earlier bootstrap additions, this is sizable enough (lifecycle + HTML template + send call) to warrant its own file rather than growing `index.ts` further.
- Sends via `strapi.plugin('email').service('email').send({...})` directly — the same low-level service the `forgotPassword` controller uses. Unlike reset-password, this isn't a Strapi/users-permissions built-in template, so there's no core-store template to configure; the HTML is ours entirely.
- Wrapped in try/catch, failures logged via `strapi.log.error` — never allowed to break or roll back the signup itself (AC27): a lifecycle subscriber's unhandled rejection would otherwise propagate into the surrounding `create()` call.
- Same Resend/nodemailer provider already configured, same design tokens and table-based inline-styled HTML as the reset-password email (`onboarding@resend.dev` sender, same layout, "Shop Now" button linking to `CLIENT_URL` instead of a reset link).
- Personalization: `event.result.firstName`, read directly off the created user record. Verified from Strapi's own Google provider source (`services/providers-registry.js`): its `authCallback` only returns `{ username, email }` from a lightweight `tokeninfo` call — no name/profile scope is ever requested — so `firstName` is only ever present for password signups; Google signups always get the generic "Welcome to WESTLINE!" heading, by design, not as a bug.
- Considered and accepted: this lifecycle fires for any `up_users` creation, including one an admin creates by hand via Content Manager — not just the two signup paths. This is Strapi's own DB-level hook behavior; documented rather than special-cased, since a welcome email to an admin-created account isn't harmful.

**Schema cleanup — removed `confirmed`, `confirmationToken`, and `blocked` (`cms`):**
- Following the full `up_users` field audit, `confirmed` was removed from `cms/src/extensions/users-permissions/content-types/user/schema.json` — verified empirically, not predicted, on a throwaway branch (`experiment/remove-confirmed-field`) before merging: registration, login, reset-password, and Google OAuth (both the initiation redirect and the exact `create()` call `providers.connect()` performs internally, including its explicit `confirmed: true`) all still succeed with no errors.
- Why it's safe: `confirmed` only gates local login/OAuth connect when `advanced.email_confirmation` is `true`; this project has that setting at its untouched default (`false`), so the one code path that reads `confirmed` never evaluates it. And Strapi's query layer silently drops `data` keys that aren't schema attributes rather than erroring, so the OAuth path's internal `confirmed: true` write is a no-op now, not a failure.
- **This closes the earlier open question from the field audit about building a real email-confirmation-at-signup feature: decided not to build it.** `confirmationToken` — the other half of that same dormant feature pair, only ever written by `sendConfirmationEmail`, which the register controller calls exclusively `if (settings.email_confirmation)` — was removed too, same dead-code reasoning, verified the same way on its own throwaway branch (`experiment/remove-confirmationtoken-field`): all four flows pass, including confirming (via direct DB query, isolated from an unrelated Resend-sandbox rejection) that `resetPasswordToken` generation/consumption is unaffected — that's a separate field entirely, just easy to conflate by name.
- `blocked` was then also removed, on a third throwaway branch (`experiment/remove-blocked-field`, branched off this spec's real branch *after* the `confirmed` removal landed — not stacked on any disposable experiment branch), same four-flow empirical verification, same result: no errors anywhere.
- **Important distinction: `blocked`'s removal was not a dead-code cleanup, unlike `confirmed`/`confirmationToken`.** `blocked` was a live, unconditionally-checked gate in Strapi's own login controller (`if (user.blocked === true) throw ...`) — it didn't error on removal only because `undefined === true` evaluates to `false`, not because nothing depended on it. Removing it is a deliberate product decision to drop the capability to block a bad account without deleting it — that capability had a real justification and this project is explicitly choosing not to need it, not discovering it was already dead.

## Out of Scope

- Custom/verified sending domain — using Resend's shared `onboarding@resend.dev` sender, which can only deliver to the Resend account's own signup email until a domain is verified in Resend.
- Email confirmation at signup (Strapi's built-in `confirmed`/`confirmationToken` feature) — decided not to build this; `confirmed` was removed from the schema entirely (see Technical Approach's "Schema cleanup" note).
- Social login providers other than Google (Facebook, GitHub, etc.) — Google only, per this change.
- Profile management, wishlist, order history — separate candidate specs in this epic.
- Seller login — separate seller-dashboard epic (the mockups' footer "Seller Login" link is not part of this spec).

## Standards Applied

- [git-workflow](../../../../../../standards/global/git-workflow.md) — branch naming (`feat/customer-account-wishlist/account-signup-login`) and commit/PR conventions.

## Changelog

| Date | Author | Type | Change | Ref |
|---|---|---|---|---|
| 2026-09-22 | Ori Chai Matan | created | Initial shaping | — |
| 2026-09-22 | Ori Chai Matan | change | Implemented T1-T17; extended Covers with the nav component, root layout, CORS config, and the users-permissions content-type extension (none of these had a home elsewhere, and AC8 needed a nav to exist) | tasks.md T1-T17 |
| 2026-09-22 | Ori Chai Matan | change | QA run: PASS WITH NOTES — T3 (live Strapi verification) not run, no local Postgres/Strapi instance available | qa-report.md |
| 2026-09-23 | Ori Chai Matan | change | Brought forgot/reset password into scope and added an admin-panel password-change lockdown, per mentor review — updated AC10, added AC11-AC18, Technical Approach, Out of Scope, Covers | — |
| 2026-09-23 | Ori Chai Matan | change | Added Google OAuth login (AC19-AC23), per your request — no Google Cloud credentials exist yet, so the feature is built to self-activate once `cms/.env` has them rather than blocking on that | — |
| 2026-09-23 | Ori Chai Matan | change | Set an absolute `server.url` in `cms/config/server.ts` (new `SERVER_URL` env var) to fix Strapi's third-party-provider warning — `buildRedirectUri` needs `server.absoluteUrl` to be real. No AC change (fixes a dependency of AC20/AC21, doesn't alter them); real Google credentials were added to `cms/.env` around the same time, unblocking T40 | — |
| 2026-09-23 | Ori Chai Matan | change | Status → done: all 42 tasks complete (T30 and T40 confirmed live by Ori); combined QA run (T31/T41): PASS WITH NOTES, 23/23 AC. Both repos committed. | qa-report.md |
| 2026-09-23 | Ori Chai Matan | change | Both commits from the row above were reverted (`git reset --soft`, at Ori's request — premature, nothing pushed) — all file changes kept, just uncommitted again | — |
| 2026-09-23 | Ori Chai Matan | change | Switched email provider from console-log to Resend (via `@strapi/provider-email-nodemailer` + Resend's SMTP relay) for real delivery — chose the official nodemailer provider over 3 low-adoption third-party Resend packages; console-log kept installed as an easy fallback. Status → in-progress (AC12 changed) | — |
| 2026-09-23 | Ori Chai Matan | change | Fixed the reset-password email's "from" address — it was silently overridden by the Users-Permissions plugin's own Email Templates core-store default (`no-reply@strapi.io`), causing Resend to reject sends with "domain not verified". `cms/src/index.ts` bootstrap now also sets `reset_password.options.from.email` idempotently | — |
| 2026-09-23 | Ori Chai Matan | change | Restyled the reset-password email to match the auth pages' design tokens — table-based, fully inline-styled HTML, system font stack (no external fonts/`<style>` blocks, which most email clients block); the `<%= URL %>?code=<%= TOKEN %>` placeholder preserved verbatim from the original template | — |
| 2026-09-23 | Ori Chai Matan | change | Added a welcome email on account creation (AC24-AC27), triggered from an `afterCreate` DB lifecycle hook so both password and Google OAuth signup share one code path | — |
| 2026-09-23 | Ori Chai Matan | change | AC24-AC27 fully verified (T51): password-signup path confirmed via clean send logs; Google OAuth first-time-signup path confirmed by Ori (generic greeting, correct styling, working Shop Now link). T52 (formal QA rerun) still open — Status stays in-progress until that runs | — |
| 2026-09-23 | Ori Chai Matan | change | Removed `confirmed` from the `up_users` schema, following the full field audit — verified empirically on a throwaway branch first (register/login/reset-password/OAuth all unaffected, since `email_confirmation` is already disabled). Closes the open question on building real email-confirmation-at-signup: decided not to. No AC affected (this field was never referenced by any AC or by our own app code) | — |
| 2026-09-23 | Ori Chai Matan | change | Removed `blocked` from the `up_users` schema too — verified empirically on a second throwaway branch (same four flows, no errors). Unlike `confirmed`, this drops a live, previously-functioning capability (blocking a bad account without deleting it) — a deliberate product decision, not dead-code removal; Ori decided this project doesn't need that capability. No AC affected | — |
| 2026-09-23 | Ori Chai Matan | change | Removed `confirmationToken` from the `up_users` schema — paired with `confirmed` under the same closed email-confirmation-feature decision (dead-code removal, not a capability drop). Verified empirically on its own throwaway branch, all four flows pass; reset-password's separate `resetPasswordToken` field confirmed unaffected via direct DB check. No AC affected | — |
| 2026-09-29 | Ori Chai Matan | change | Handed `frontend/src/components/site-nav.tsx` to storefront-catalog/2026-09-29-site-nav (AC8's account-icon behavior stays owned here); backfilled Covers to the real paths after the web→frontend rename and the move of auth under `/auth/` (`/signup` → `/auth/register`, `web/src/app/(auth)/**` → `frontend/src/app/[locale]/auth/**`, `web/src/app/layout.tsx` → `frontend/src/app/[locale]/layout.tsx`, `cms/src/welcome-email.ts` → `cms/src/bootstrap/welcome-email.ts`); dropped `web/src/app/api/auth/oauth/**` (covered by `frontend/src/app/api/auth/**`) and `cms/providers/console-log/**` (no longer exists) | 2026-09-29-site-nav |

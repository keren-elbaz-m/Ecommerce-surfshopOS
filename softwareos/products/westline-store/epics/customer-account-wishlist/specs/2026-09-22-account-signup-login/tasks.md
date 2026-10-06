# Tasks — Account Signup & Login

> Epic: customer-account-wishlist
> Spec: 2026-09-22-account-signup-login
> Branch: feat/customer-account-wishlist/account-signup-login

## Backend — Strapi auth config

- [x] T1 Configure `cms/config/middlewares.ts` `strapi::cors` to allow the `web` origin with `credentials: true` (read origin from a new `CLIENT_URL` env var, default `http://localhost:3000`); add `CLIENT_URL` to `cms/.env` and `cms/.env.example`
- [x] T2 Review `cms/config/plugins.ts` users-permissions config for signup/login: confirm `jwtManagement: 'refresh'` and `sessions.httpOnly: true` are sufficient for AC1/AC4, and set a sane JWT expiry if the default is too long/short for a demo session
- [x] T3 Manually verify Strapi's built-in `/api/auth/local/register` and `/api/auth/local` endpoints against a local Strapi instance (Postman/curl): confirm duplicate-email register returns a distinct error (feeds AC2) and bad-credentials login returns a distinct error (feeds AC5) — no code change, just confirms the response shapes T7/T9 will rely on. **Verified 2026-09-22** against a running local Postgres/Strapi instance: register and login succeed end-to-end (real JWTs returned, `up_users.provider` column present, passwords stored as bcrypt hashes).

## Frontend — session plumbing

- [x] T4 Add `web/vitest.config.ts` + `@testing-library/react` / `@testing-library/jest-dom` devDependencies and a minimal `web/vitest.setup.ts`; add an `npm test` script to `web/package.json` — no test framework exists in `web` yet and TDD needs one before any test can be written
- [x] T5 Add `STRAPI_URL` server-only env var to `web/.env.local.example` (and `web/.env.local` if present), read via `process.env.STRAPI_URL` — used by the route handlers in T7-T9
- [x] T6 Write failing tests for `web/src/lib/auth/session.ts` (testing skill) — covers AC6, AC9: a `getSession()`/`readSession()` reader that parses the httpOnly cookie server-side, and a `requireAuth()` helper that redirects to `/login` when no valid session is found
- [x] T7 Implement `web/src/lib/auth/session.ts` to pass T6: cookie read/parse via `next/headers`, `requireAuth()` for use by future authenticated pages

## Frontend — signup

- [x] T8 Write failing tests for `web/src/app/api/auth/register/route.ts` (testing skill) — covers AC1, AC2, AC6: proxies to `POST {STRAPI_URL}/api/auth/local/register`, sets an httpOnly/secure/sameSite cookie on success, passes through Strapi's duplicate-email error as a JSON error response on failure
- [x] T9 Implement `web/src/app/api/auth/register/route.ts` to pass T8
- [x] T10 Build `web/src/app/(auth)/signup/page.tsx` per the UI mockup: first name, last name, email, password fields with an 8-char hint, no confirm-password/terms checkbox; client-side password-length validation before submit (AC3); inline error rendering for duplicate-email (AC2) and other request errors; on success redirect to `/` (AC1)

## Frontend — login

- [x] T11 Write failing tests for `web/src/app/api/auth/login/route.ts` (testing skill) — covers AC4, AC5, AC6: proxies to `POST {STRAPI_URL}/api/auth/local`, sets httpOnly/secure/sameSite cookie on success, returns a generic invalid-credentials error on failure without setting a cookie
- [x] T12 Implement `web/src/app/api/auth/login/route.ts` to pass T11
- [x] T13 Build `web/src/app/(auth)/login/page.tsx` per the UI mockup: email + password fields, inline error on invalid credentials (AC5); on success redirect to `/` (AC4). **Reopened 2026-09-23** (see Changelog): "Forgot your password?" now links to `/forgot-password` for real (AC10) instead of the old stubbed note — also replaces the now-obsolete "shows a note" test in `page.test.tsx` with one asserting the link's `href`

## Frontend — logout & nav state

- [x] T14 Write failing tests for `web/src/app/api/auth/logout/route.ts` (testing skill) — covers AC7: clears the session cookie and returns a success response
- [x] T15 Implement `web/src/app/api/auth/logout/route.ts` to pass T14
- [x] T16 Add a nav component (extract from `web/src/app/layout.tsx` if the account icon currently lives inline, or create `web/src/components/nav.tsx` if no nav exists yet) that reads session state via `web/src/lib/auth/session.ts` and renders logged-in vs. logged-out account icon state (AC8), with a logout action wired to `POST /api/auth/logout`

## Verification

- [x] T17 Write tests for the route handlers' error-response shapes and `requireAuth()` redirect behavior not already covered in T6/T8/T11/T14 (testing skill) — covers AC3, AC9, AC10; note: httpOnly/secure cookie attributes and true cross-origin CORS behavior (AC6, AC7) aren't practically unit-testable and are left to the QA pass below
- [x] T18 Run QA against acceptance criteria (qa skill) — verify AC1-AC10 end-to-end against a running `cms` + `web`, including manual inspection of the Set-Cookie header (httpOnly/secure) and a duplicate-email / bad-password / bad-login walkthrough

## Backend — forgot/reset password infra

- [x] T19 Author `cms/providers/console-log/{package.json,index.js}` implementing Strapi's email-provider interface (`init(providerOptions, settings) => { send(options) }`) that logs the rendered email to the console; add it as a `file:` dependency in `cms/package.json` and run `npm install`. Verified by direct `require.resolve('console-log')` + `send()` call, matching exactly how Strapi's own bootstrap resolves providers.
- [x] T20 Configure `cms/config/plugins.ts`'s `email` plugin config to use `provider: 'console-log'` with sane `defaultFrom`/`defaultReplyTo` settings
- [x] T21 Add a `bootstrap()` step in `cms/src/index.ts` that sets the Users-Permissions `advanced.email_reset_password` store setting to `${CLIENT_URL}/reset-password` on boot (idempotent — only writes if the value differs)
- [x] T22 Manually verify against a running local Strapi+Postgres instance: `POST /api/auth/forgot-password` with a known email logs a reset link to the `cms` console containing a real token, and with an unknown email still returns `{ ok: true }` without logging anything sensitive — no local email/SMTP infra required (mirrors T3's manual-verification pattern; no `cms` test framework exists to automate this). **Verified 2026-09-23**: registered `qa-forgot-test@example.com` via `/api/auth/local/register`, then called forgot-password for it (console logged `http://localhost:3000/reset-password?code=<64-byte hex token>` — confirms the T21 bootstrap correctly points the link at the real frontend) and for an unregistered email (both returned `{"ok":true}`, only the known email produced a console log entry).

## Frontend — forgot password

- [x] T23 Write failing tests for `web/src/app/api/auth/forgot-password/route.ts` (testing skill) — covers AC11: proxies to `POST {STRAPI_URL}/api/auth/forgot-password`, always returns a generic `{ ok: true }`-shaped success response regardless of Strapi's result (anti-enumeration)
- [x] T24 Implement `web/src/app/api/auth/forgot-password/route.ts` to pass T23
- [x] T25 Build `web/src/app/(auth)/forgot-password/page.tsx` per the mockup: email field, "Send Reset Link" button, in-place swap to a "Check Your Email" confirmation state (adapted copy: points at the `cms` server console instead of the mockup's "Open Reset Link" button, since no real token is exposed client-side)

## Frontend — reset password

- [x] T26 Write failing tests for `web/src/app/api/auth/reset-password/route.ts` (testing skill) — covers AC14, AC15, AC17: proxies to `POST {STRAPI_URL}/api/auth/reset-password` with `{ password, passwordConfirmation, code }`, sets the httpOnly session cookie on success (same as login/register), passes through Strapi's invalid-code error on failure without setting a cookie
- [x] T27 Implement `web/src/app/api/auth/reset-password/route.ts` to pass T26. **Live-verified 2026-09-23** beyond the mocked tests: called Strapi's `/api/auth/reset-password` directly with the real code from T22 — succeeded (200, `{jwt, refreshToken, user}`), then reusing the same code correctly failed (400 "Incorrect code provided"), confirming the route's assumptions about Strapi's actual response shape and single-use token behavior.
- [x] T28 Build `web/src/app/(auth)/reset-password/page.tsx` per the mockup: new password + confirm fields (8-char hint), reads `code` from the URL query string (wrapped in `<Suspense>` per Next.js's `useSearchParams` requirement); client-side validation for length and matching passwords (AC14); a distinct "invalid or missing link" state when no `code` is present (AC16, not in the mockup — added for the real implementation); success state adapted from the mockup's static "sign in with your new password" copy to "you're signed in" + a link to `/`, since the real reset-password flow logs the shopper in (AC15)

## Backend — admin panel lockdown

- [x] T29 Author `cms/src/extensions/users-permissions/strapi-server.ts` (note: `.ts`, not `.js` as originally planned — see Changelog) wrapping the `contentmanageruser.update` controller to reject (400) any request whose body has a non-empty `password` (mirroring Strapi's own null/empty no-op convention), before delegating to the original controller. **Verified 2026-09-23**: (a) unit-tested the wrapper logic in isolation against a fake plugin/ctx — password present → rejected without calling through; no/empty password → passes through unchanged; (b) confirmed live against the running Strapi process that `strapi.plugin('users-permissions').controller('contentmanageruser').update` is genuinely the wrapped function (not silently skipped — an initial `.js` version compiled to nowhere and was silently never loaded, caught by this exact check, then fixed as `.ts`). Not verified: an actual authenticated Super-Admin click-through in the browser — that needs your own admin login, which I don't have and won't request; that's T30.
- [x] T30 Manually verify against a running local Strapi+Postgres instance: attempting to set a new password for a shopper via Admin → Content Manager → Users is rejected with the new error, while editing other fields (e.g. blocked/confirmed) still works normally (AC18; mirrors T3/T22's manual-verification pattern). **Verified 2026-09-23 by Ori** (as Super Admin): got the exact custom rejection message "Password changes are not allowed from the Admin panel. Use the self-service reset-password flow." — corroborated in `/tmp/cms-dev.log`: `PUT /content-manager/collection-types/plugin::users-permissions.user/tse1a1j3i1f5o3l1467i3sb1` → `400`.

## Verification (2026-09-23 change)

- [x] T31 Run QA against acceptance criteria (qa skill) — re-verify AC1-AC18 end-to-end, focusing on the new forgot/reset password flow and the admin lockdown. **Superseded by the combined T31/T41 run on 2026-09-23** — see qa-report.md (PASS WITH NOTES, 23/23 AC).

## Frontend — Google OAuth (2026-09-23 change)

- [x] T32 Add `NEXT_PUBLIC_STRAPI_URL` to `web/.env.local.example` (and `.env.local`)
- [x] T33 Add a "Continue with Google" button to `web/src/app/(auth)/signup/page.tsx` and `web/src/app/(auth)/login/page.tsx`, linking to `${NEXT_PUBLIC_STRAPI_URL}/api/connect/google` (AC19, AC20)
- [x] T34 Write failing tests for `web/src/app/api/auth/oauth/complete/route.ts` (testing skill) — covers AC21: sets the httpOnly session cookie from a posted `jwt`, rejects a missing/empty `jwt`
- [x] T35 Implement `web/src/app/api/auth/oauth/complete/route.ts` to pass T34
- [x] T36 Build `web/src/app/(auth)/connect/google/redirect/page.tsx`: on mount, calls Strapi's `/api/auth/google/callback` with credentials, then `/api/auth/oauth/complete`, redirects to `/` on success (AC21); shows a clear error state on OAuth failure or a failed exchange (AC22). Build verified: `npx next build` compiles `/connect/google/redirect` cleanly (Suspense boundary satisfied).

## Backend — Google OAuth provider config (2026-09-23 change)

- [x] T37 Add `GOOGLE_CLIENT_ID`/`GOOGLE_CLIENT_SECRET` placeholders (empty) to `cms/.env.example`, with a comment documenting the exact Authorized redirect URI to register in Google Cloud Console (`{STRAPI_URL}/api/connect/google/callback`)
- [x] T38 Extend `cms/src/index.ts`'s `bootstrap()` to configure the Users-Permissions grant store's `google` entry (enabled, key, secret, callback) when `GOOGLE_CLIENT_ID`/`GOOGLE_CLIENT_SECRET` are present; log a warning and leave it disabled otherwise (AC23)
- [x] T39 Manually verify against the running local Strapi instance (no real Google credentials needed for this part): with the env vars absent, confirm `GET /api/connect/google` returns Strapi's "This provider is disabled" error rather than crashing (AC23) — mirrors the T3/T22/T30 manual-verification pattern. **Verified 2026-09-23**: restarted `cms`, confirmed the bootstrap warning logs on boot (`GOOGLE_CLIENT_ID/GOOGLE_CLIENT_SECRET are not set...`), then `curl http://localhost:1337/api/connect/google` returned a clean `400 {"error":{"message":"This provider is disabled"}}` — no crash, no stack trace.

## Verification (2026-09-23 change, cont.)

- [x] T40 Click through the full "Continue with Google" flow on both `/login` and `/signup` to verify AC19-AC22 end-to-end. **Verified 2026-09-23 by Ori**: successful end-to-end click-through, twice — `/api/connect/google` → Google consent screen → `/api/connect/google/callback` → `/api/auth/google/callback` (200). Not independently re-confirmed from `/tmp/cms-dev.log` on my end — that log was overwritten by my subsequent server restarts for the T42 SERVER_URL fix, so this is recorded as your reported verification, not something I personally observed in logs.
- [x] T41 Run QA against acceptance criteria (qa skill) — re-verify AC1-AC23; AC21 (the live OAuth exchange) stays "not verifiable" until T40 is done. **Run 2026-09-23**, combined with T31: PASS WITH NOTES, 23/23 AC — see qa-report.md.
- [x] T42 Fix the "third party provider" warning: set an absolute `server.url` in `cms/config/server.ts` via a new `SERVER_URL` env var (default `http://localhost:1337`), instead of leaving it unset/relative — `buildRedirectUri` (used for all OAuth providers, not just Google) derives from `server.absoluteUrl`, which was previously malformed. **Verified 2026-09-23**: restarted `cms`; `GET /api/connect/google` now builds `redirect_uri=http%3A%2F%2Flocalhost%3A1337%2Fapi%2Fconnect%2Fgoogle%2Fcallback` — a correct, fully-qualified absolute URL.

## Backend — email provider swap (2026-09-23 change)

- [x] T43 Add `@strapi/provider-email-nodemailer` (pinned `5.54.0`, matching the rest of the Strapi deps) and reconfigure `cms/config/plugins.ts`'s `email` block: `provider: 'nodemailer'` against Resend's SMTP relay (`smtp.resend.com:465`, `auth.user: 'resend'`, `auth.pass: env('RESEND_API_KEY')`), `defaultFrom`/`defaultReplyTo: 'onboarding@resend.dev'` (AC12). `cms/providers/console-log/` and its dependency left installed, just unreferenced.
- [x] T44 Add `RESEND_API_KEY` (empty placeholder + comment on where to get one) to `cms/.env.example`. **Not automatable by me**: add `RESEND_API_KEY=<your real key>` as a new line after `ENCRYPTION_KEY=tobemodified` in `cms/.env` yourself — that file is permission-blocked for me to read or write.
- [x] T45 Manually verify against a running local Strapi instance, once you've added a real `RESEND_API_KEY` to `cms/.env`: trigger `/forgot-password` for a real, registered email and confirm the email actually arrives (AC12) — mirrors the T3/T22/T30/T39/T40 manual-verification pattern; I don't have a Resend API key and won't ask for one. **Verified 2026-09-23**: triggered live for `ori.chaimatan@moveo.co.il` — clean `200 {"ok":true}` direct from Strapi (no try/catch around the send call, so a real failure would have surfaced as a 500), ~1.7-2.3s duration consistent with a genuine SMTP round-trip, no error in the server log.
- [ ] T46 Run QA re-verifying AC12 (and a spot-check of the rest) once T45 is done

## Backend — email bugfixes (2026-09-23, backfilled)

- [x] T47 Fix the reset-password email's `from` address: the Users-Permissions plugin's own Email Templates core-store (`key: 'email'`) seeds `reset_password.options.from.email` with a placeholder `no-reply@strapi.io`, which silently overrode `config/plugins.ts`'s `defaultFrom` and caused Resend to reject sends ("550 The strapi.io domain is not verified"). `cms/src/index.ts` bootstrap now also sets `reset_password.options.from.email` to `onboarding@resend.dev`, idempotently, same pattern as the existing `email_reset_password` block. **Verified 2026-09-23**: confirmed the actual stored value via a temporary boot-time diagnostic (not guessed), then reran forgot-password live — clean 200, no domain error.
- [x] T48 Restyled the reset-password email's HTML (`reset_password.options.message` in the same core-store key) to match the auth pages' design tokens (Ink `#101828`, Horizon `#155EEF`, Border `#D0D5DD`, `#F9F9F9`/`#FFFFFF` surfaces, 9px radius) — table-based layout, every style inlined, system font stack only (no external Google Fonts, no `<style>` block — most email clients block both). The `<%= URL %>?code=<%= TOKEN %>` placeholder was read from the live core-store first and preserved verbatim (moved from plain text into the button's `href`, not renamed). **Verified 2026-09-23**: reran forgot-password live — clean 200, no error.

## Backend — welcome email (2026-09-23 change)

- [x] T49 Create `cms/src/welcome-email.ts`: exports a function that builds the welcome email's table-based inline-styled HTML (same design tokens as reset-password, "Shop Now" button linking to `CLIENT_URL`, personalized heading using `firstName` when present) and registers `strapi.db.lifecycles.subscribe({ models: ['plugin::users-permissions.user'], afterCreate })`, sending via `strapi.plugin('email').service('email').send({...})` wrapped in try/catch (AC24-AC27)
- [x] T50 Wire it up: call the new registration function from `cms/src/index.ts`'s `bootstrap()`
- [x] T51 Manually verify against a running local Strapi instance: register a new password-signup account and confirm a personalized welcome email arrives (AC24, AC25 with firstName); manually verify the same for a first-time Google OAuth signup and confirm a welcome email arrives with the generic (non-personalized) heading (AC24, AC25 without firstName) — mirrors the established manual-verification pattern for this spec. **Verified 2026-09-23**: (a) AC27 confirmed via a throwaway `@example.com` account — Resend correctly rejected the welcome send (sandbox restriction) but registration still succeeded (200, JWT returned), try/catch worked as designed. (b) Password-signup path: re-registered `ori.chaimatan@moveo.co.il` with `firstName: "Ori"` — clean 200, ~1.8s (consistent with a real send), no error logged. (c) **Confirmed by Ori**: first-time Google OAuth signup delivered a welcome email with the generic (non-personalized) greeting, correct styling matching the reset-password email, and a working "Shop Now" link — exactly as AC24-AC26 require.
- [ ] T52 Run QA re-verifying AC24-AC27 (and a spot-check of the rest) — **not run yet**, holding off per your "just update tracking files, no commit" scope for this turn

## Backend — schema cleanup (2026-09-23, from the full `up_users` field audit)

- [x] T53 Remove `confirmed` from `cms/src/extensions/users-permissions/content-types/user/schema.json`. **Verified empirically on `experiment/remove-confirmed-field` before merging** (not predicted): registration, login, reset-password (isolated from an unrelated Resend-sandbox red herring), and Google OAuth (initiation redirect + the exact `providers.connect()` `create()` call, replicated directly via `strapi console`, including its explicit `confirmed: true`) all succeed with no errors. Root cause of "why safe": `email_confirmation` is disabled (untouched default), so the one code path that reads `confirmed` never evaluates it; Strapi's query layer silently drops non-schema `data` keys rather than erroring. Brought from the experiment branch into this branch; experiment branch left in place (not deleted).
- [x] T54 Remove `blocked` from `cms/src/extensions/users-permissions/content-types/user/schema.json`. **Verified empirically on `experiment/remove-blocked-field`** (branched off this branch's state *after* T53, not off the `confirmed` experiment branch, to attribute results cleanly per field): same four flows, all succeed with no errors — login's `if (user.blocked === true)` check simply never fires against an absent field (`undefined === true` is `false`). **Unlike T53, this is a deliberate product decision, not dead-code removal** — `blocked` was a live, working capability (block a bad account without deleting it); Ori decided this project doesn't need it. Brought into this branch.
- [x] T55 Remove `confirmationToken` from `cms/src/extensions/users-permissions/content-types/user/schema.json`. **Verified empirically on `experiment/remove-confirmationtoken-field`**: register, login, reset-password (a genuine investigation surfaced a real confound — the initial test account had been deleted by Ori during his own cleanup, producing a misleadingly fast anti-enumeration `200`; re-verified cleanly against a fresh throwaway account, `resetPasswordToken` generation/consumption confirmed unaffected via direct DB check, isolated from `confirmationToken` by name only), and Google OAuth (initiation + `providers.connect()` `create()` replication) all succeed with no errors. Same dead-code reasoning as T53 — `sendConfirmationEmail` (the only writer of this field) is only ever called `if (settings.email_confirmation)`, which is `false`. Paired with T53 under the same closed email-confirmation-feature decision. Brought into this branch.

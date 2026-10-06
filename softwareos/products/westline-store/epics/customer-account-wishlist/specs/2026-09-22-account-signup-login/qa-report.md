# QA Report — Account Signup & Login

> Spec: spec.md
> Date: 2026-09-23 · QA by: Ori Chai Matan · Mode: setup
> Verdict: PASS WITH NOTES
> Stale: spec changed 2026-09-23 (Acceptance Criteria: AC12 reworded for the Resend email-provider swap; AC24-AC27 added for the welcome email) — rerun the qa skill

## Acceptance Criteria

| AC | Criterion | Result | Evidence |
|---|---|---|---|
| AC1 | Signup at `/signup` with valid fields → logged in, redirected home | pass | `web/src/app/api/auth/register/route.test.ts`; live-verified against real Strapi (register succeeded, `firstName`/`lastName` persisted) |
| AC2 | Duplicate email rejected with clear inline error | pass | `register/route.test.ts` ("passes through a duplicate-email error"); error rendered via `signup/page.tsx` |
| AC3 | Password under 8 chars rejected before reaching Strapi | pass | `web/src/app/(auth)/signup/page.test.tsx` (client) + `register/route.test.ts` (server, defense in depth) |
| AC4 | Login at `/login` with valid credentials → redirected home | pass | `web/src/app/api/auth/login/route.test.ts` |
| AC5 | Invalid credentials → generic inline error, no session | pass | `login/route.test.ts` |
| AC6 | Session stored as httpOnly/secure cookie, never exposed to client JS | pass | `web/src/lib/auth/session.ts` (`getSessionCookieOptions`); asserted `httpOnly: true` in register/login/reset-password/oauth-complete route tests |
| AC7 | Logout clears session cookie, returns logged-out state | pass | `web/src/app/api/auth/logout/route.test.ts` |
| AC8 | Nav account icon reflects logged-in/out state | pass (inspection) | `web/src/components/site-nav.tsx` — reads `getSession()`, renders `LogoutButton` vs. Sign In link |
| AC9 | `requireAuth()` redirects unauthenticated requests to `/login` | pass | `web/src/lib/auth/session.test.ts` |
| AC10 | "Forgot your password?" navigates to `/forgot-password` (real, not stubbed) | pass | `web/src/app/(auth)/login/page.test.tsx` (asserts `href="/forgot-password"`) |
| AC11 | `/forgot-password` shows a generic confirmation regardless of email existence | pass | `web/src/app/api/auth/forgot-password/route.test.ts`; live-verified: known + unknown email both returned `{ok:true}`, only known email logged |
| AC12 | Matching account → console-log provider logs the reset link, no real email sent | pass | Live-verified: registered `qa-forgot-test@example.com`, console logged `http://localhost:3000/reset-password?code=<64-byte hex>` |
| AC13 | `/reset-password?code=<token>` shows the set-new-password form | pass (inspection) | `web/src/app/(auth)/reset-password/page.tsx` — renders the form when `code` is present; no dedicated automated test for this specific branch |
| AC14 | Mismatched password/confirmation or under 8 chars → inline error before Strapi | pass | `web/src/app/api/auth/reset-password/route.test.ts` (server); client validation inspected in `reset-password/page.tsx` |
| AC15 | Valid new password + valid code → password updated, shopper logged in, success state | pass | `reset-password/route.test.ts`; live-verified against real Strapi (real code succeeded, returned `{jwt, refreshToken, user}`) |
| AC16 | Missing code → "link is invalid or expired" state, no request attempted | pass (inspection) | `reset-password/page.tsx`'s `if (!code)` branch; no dedicated automated test |
| AC17 | Invalid/expired/reused code → inline error, no password change, no session | pass | `reset-password/route.test.ts`; live-verified: reusing the same code after success correctly failed with "Incorrect code provided" |
| AC18 | Password cannot be changed via Admin Content Manager, any role including Super Admin | pass | `cms/src/extensions/users-permissions/strapi-server.ts`, unit-verified in isolation (password present → rejected; empty/absent → passes through); live-verified as Super Admin — exact rejection message reproduced, corroborated in server log (`PUT .../users-permissions.user/... 400`) |
| AC19 | "Continue with Google" button on both `/login` and `/signup` | pass (inspection) | `web/src/app/(auth)/_components/google-button.tsx`, used in both pages |
| AC20 | Clicking it navigates to Strapi's `/api/connect/google` | pass (inspection) | `google-button.tsx`'s `href` |
| AC21 | Successful Google sign-in → httpOnly session cookie, redirected home | pass | `web/src/app/api/auth/oauth/complete/route.test.ts` (3 tests); live-verified end-to-end by the user, twice (`/api/connect/google` → Google consent → `/api/connect/google/callback` → `/api/auth/google/callback` 200) — not independently re-observed in logs on my end (overwritten by a later server restart), recorded as the user's reported verification |
| AC22 | Failed/cancelled Google sign-in → clear error state, not a broken page | pass (inspection) | `web/src/app/(auth)/connect/google/redirect/page.tsx` reads `error`/`error_description` query params Strapi appends on failure; no live failure-path click-through was performed (only the success path was exercised) |
| AC23 | Without Google credentials, button renders but fails gracefully server-side | pass | Live-verified: with env vars absent, boot logged the expected warning and `GET /api/connect/google` returned a clean `400 "This provider is disabled"` |

23/23 pass. Three carry an inspection-only or partially-inspection caveat (AC13, AC16, AC22) and one relies on the user's reported (not independently re-observed) live verification (AC21) — see Notes.

## Checks Run

- build: `npx next build` (web) — pass (all 13 routes compiled, including `/forgot-password`, `/reset-password`, `/connect/google/redirect`)
- lint: `npx next lint` (web) — pass, 0 warnings/errors
- typecheck: `npx tsc --noEmit` (web) — pass
- typecheck: `npx tsc --noEmit` (cms) — pass
- tests: `npx vitest run` (web) — pass, 21/21 (9 files)

cms still has no test framework (unchanged from the prior report) — all cms-side verification is either live (T3, T22, T27, T30, T39) or ad-hoc unit checks run directly via `node -e` (T19, T29), not part of a committed suite.

## Issues Found

(none) — no defects found this run. The two prior defects (Issue 1 from the previous report — content-type schema replacement and missing `allowedFields`) were fixed and re-verified before this report.

## Notes

- **AC13/AC16 have no dedicated automated test** for their specific branches (code-present-shows-form, code-missing-shows-error) — covered by code inspection only. Low risk (simple conditional rendering, and the surrounding logic is TDD-covered), but worth a lightweight component test if this page changes again.
- **AC21's live verification is the user's report, not something I independently re-confirmed from logs** — the `cms` server log that would have shown the callback sequence was overwritten by a later restart (done to verify the unrelated T42 `SERVER_URL` fix) before I could check it. Every other piece of the OAuth chain (the route handler, the redirect page's logic, the bootstrap provider config, the disabled-by-default behavior) is independently verified either by test or by me directly.
- **AC22 (OAuth failure path) was never actually exercised** — only the success path was live-tested. The error-state code is straightforward (reads two query params Strapi documents appending on failure) but is inspection-only.
- Mode is `setup`: none of the above caveats change the verdict from PASS WITH NOTES — in `production` mode, AC13/AC16/AC22 (no test evidence) and AC21 (unconfirmed-by-me live evidence) would need to be closed out first.
- Hygiene: 40/42 tasks `[x]`; the remaining two (T31, T41) are this QA run itself and will be closed by it. No PR opened yet (`gh` isn't available in this environment to check for one, and no `> PR:` line is recorded in `tasks.md`).
- Both repos (`Ecommerce-surfshop` and `Ecommerce-surfshopOS`) were committed for the first time this session, covering all of T1-T42.

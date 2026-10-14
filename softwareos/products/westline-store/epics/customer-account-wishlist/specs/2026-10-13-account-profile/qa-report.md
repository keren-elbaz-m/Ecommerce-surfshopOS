# QA Report — Account Profile

> Spec: spec.md
> Date: 2026-10-14 · QA by: Ori Chai Matan · Mode: setup
> Verdict: PASS WITH NOTES

## Setup

- **Branch:** `feat/customer-account-wishlist/account-profile`. It includes hotfix `30-day-sessions`, merged as `079b33c`.
- **Servers:** cms on :1337 and frontend `next dev` on :3000.
- **Browser runs:** headless Chromium (Playwright).
- **Test data:** two throwaway `qa-acct-*@example.com` users, deleted afterwards (`DELETED {"count":2}`, no carts).

## Acceptance Criteria

| AC | Criterion | Result | Evidence |
|---|---|---|---|
| AC1 | Signed-out `/account` → `/auth/login?next=%2Faccount`, back after login | pass | **Runtime:** redirected to `/auth/login?next=%2Faccount`, then `/account` after login. **Test:** `__tests__/app/[locale]/account/page.test.tsx` (guest + rejected session) |
| AC2 | Breadcrumb "Home / My Account", h1, "Welcome back, {first} {last}." | pass | **Runtime:** breadcrumb `Home/My Account`, h1 `My Account`, `Welcome back, Qa Tester.` **Test:** `AccountPage.test.tsx` |
| AC3 | Sign Out: spinner + "Signing out…", disabled; session cleared; Home | pass | **Runtime:** `["Signing out…", disabled]`, then `/` with no `westline_session` cookie; `/account` then redirects to login. **Test:** `SignOutButton.test.tsx` |
| AC4 | Two tabs, one active, horizon underline, `replaceState` hash | pass | **Runtime:** aria-selected `[true,false]`, underline `rgb(21,94,239)`, click → `#details` with unchanged `history.length`, only `account-panel-details` visible. **Tests:** `use-account-tab.test.tsx`, `AccountPage.test.tsx` |
| AC5 | Hash routing | pass | **Runtime:** `#account`/`#details` → Account Details; `#wishlist`/`#orders`/`#zzz` → My Account. Scroll to `#orders` checked on 2026-10-13 (scrollY 257 at 375×500). **Test:** `use-account-tab.test.tsx` (13 cases) |
| AC6 | First/Last/Email prefilled, no Phone | pass | **Runtime:** labels `First name, Last name, Email`, prefilled values. **Test:** `AccountDetailsForm.test.tsx` |
| AC7 | Client validation, inline, no request | pass | **Runtime:** "Enter your first name." + "Enter a valid email address.", 0 requests to `/api/account/profile`; a 51-char name → "Use at most 50 characters." **Tests:** `validate-profile.test.ts`, `AccountDetailsForm.test.tsx` |
| AC8 | "Saving…" disabled → "Profile updated." ~2.5s, welcome line updates | pass | **Runtime:** `["Saving…", true]`, then `Save Changes` + `Welcome back, Casey Morgan.`; note gone after 2.7s. **Test:** `AccountDetailsForm.test.tsx` (fake timers) |
| AC9 | Persists to own Strapi user, `username` = email | pass | **Runtime:** reload shows `Casey / Morgan`; 2026-10-13 curl showed `username` following `email`. **Tests:** `cms/tests/account/account-store.test.ts`, `account.test.ts` (`updateMe`) |
| AC10 | Server re-validates, whitelists, own account only | pass | **Runtime (curl as A):** body with B's `id`, `password`, `username`, `role` → 200 changing only A's names; A's old password still logs in; B unchanged; invalid body → 400 with fields; core `PUT /api/users/:id` → 403. **Tests:** `account-logic.test.ts`, `account-controller.test.ts`, `route.test.ts` |
| AC11 | Taken email (any case) → inline error, nothing saved | pass | **Runtime:** B's email upper-cased → "An account with this email already exists." under Email. **Tests:** `account-store.test.ts`, `AccountDetailsForm.test.tsx` |
| AC12 | Other failure → error note, values kept; expired session → sign-in | pass | **Runtime:** forced 502 → `form [role=alert]` "Couldn't save your changes. Please try again." in `rgb(192,57,43)`, last name kept, button enabled. Garbage session cookie → `/auth/login?next=%2Faccount`. **Test:** `AccountDetailsForm.test.tsx` |
| AC13 | Wishlist then Order History, "0 saved", both empty states + links | pass | **Runtime:** h2 order `Wishlist, Order History`; `0 saved` and the orders empty state visible; links `/products/category/surfboards`, `/products/category/clothing`. **Test:** `AccountOverview.test.tsx` |
| AC14 | Footer "© 2026 WESTLINE. Find Your Line." | pass | **Runtime:** footer text matches. **Test:** `AccountPage.test.tsx` |
| AC15 | Responsive 1280/900/375 | pass | **Runtime:** no horizontal overflow at any width; name row `row`/`row`/`column`; gutter 24/24/16px; tablist `overflow-x:auto` |

## Checks Run

- tests (frontend): `npx vitest run` — pass (83 files, 746 tests)
- typecheck (frontend): `npx tsc --noEmit` — pass
- lint (frontend): `npm run lint` — pass (1 pre-existing warning, `app/[locale]/layout.tsx:32` custom font, untouched by this spec)
- build (frontend): `npm run build` — pass. `/[locale]/account` is listed as ● like `/checkout` (from the layout's `generateStaticParams`); `force-dynamic` still gates per request, as the runtime 307 confirms
- typecheck (cms): `npx tsc --noEmit` — pass
- tests (cms): `npm test` — pass (16 files, 252 tests)

## Issues Found

(none)

## Notes

- **Hygiene:** no PR yet. One PR carries account-profile + wishlist + the home hero fix, opened after review; `> PR:` is recorded then. All other tasks are `[x]`. T22 is closed by this run.
- **Process note:** running `npm run build` while `next dev` was up overwrote the shared `.next` and broke the dev server (500s on every page). The dev server was restarted with a clean `.next` before the runtime checks. This doesn't affect the spec, but don't build alongside a running dev server.
- **Hotfix:** the 30-day-sessions hotfix (owned by account-signup-login) is now in this branch. Its regression tests (`session.test.ts`) pass as part of the 746.

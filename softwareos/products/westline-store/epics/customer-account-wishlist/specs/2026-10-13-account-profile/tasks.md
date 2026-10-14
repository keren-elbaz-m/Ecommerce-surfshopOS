# Tasks — Account Profile

> Epic: customer-account-wishlist
> Spec: 2026-10-13-account-profile
> Branch: feat/customer-account-wishlist/account-profile

## Backend — profile endpoint (cms)

- [x] T1 Add `cms/src/api/account/account-logic.ts`: `validateProfile(body)` that whitelists firstName/lastName/email, trims them, checks required, max 50 and email format, and lowercases the email (AC10)
- [x] T2 Add `cms/src/api/account/{routes,controllers,services}/account.ts`: `PATCH /account/profile` (`auth: false`), resolves the user via `createIdentify` from `api/cart/identity.ts` (401 unless `kind: 'user'`), returns 409 `email-taken` on a case-insensitive email clash with another user, and updates `{ firstName, lastName, email, username: email }` on the caller's own id (AC9-AC11)
- [x] T3 Manually verify against local Strapi with curl: own update succeeds; a duplicate email gives 409; no or invalid JWT gives 401; extra fields (`role`, `password`, `id`) are ignored; another user's record is unchanged. **Verified 2026-10-13** against the running local Strapi with two throwaway `@example.com` users (deleted afterwards): own update → 200 with the email lowercased and `username` following it; mixed-case duplicate → 409 `email-taken`; invalid → 400 with fields; no/garbage JWT → 401; smuggled `password`/`id`/`username`/`role` ignored (old password still logs in, username = email); user B unchanged

## Frontend — data + validation

- [x] T4 Write failing tests for `lib/account/validate-profile.ts` (testing skill): AC7, AC10
- [x] T5 Implement `lib/account/validate-profile.ts` + `validate-texts.ts` to pass T4
- [x] T6 Write failing tests for `updateMe` in `__tests__/lib/strapi/account.test.ts` (added as a second `describe` in checkout's existing test file): the 200, 401, 409 and network-failure mappings
- [x] T7 Implement `updateMe` in `lib/strapi/account.ts` to pass T6
- [x] T8 Add `ACCOUNT_HREF` and `ACCOUNT_PROFILE_API_HREF` to `lib/routes.ts`, derive `WISHLIST_HREF` from `ACCOUNT_HREF`, and update `__tests__` if routes are asserted
- [x] T9 Write failing tests for `app/api/account/profile/route.ts`: no cookie → 401; invalid body → 400 with fields; 409 → `fields.email`; Strapi failure → 502; success → 200 with the profile (AC9-AC12)
- [x] T10 Implement `app/api/account/profile/route.ts` to pass T9

## Frontend — page + components

- [x] T11 Write failing tests for `components/account/use-account-tab.ts`: every hash mapping, `hashchange` handling, `replaceState` on tab change, and scroll on wishlist/orders (AC4, AC5)
- [x] T12 Implement `use-account-tab.ts` to pass T11
- [x] T13 Add `components/account/account-texts.ts` with every string from the design plus the two flagged gap strings
- [x] T14 Write failing tests for `AccountDetailsForm.tsx` (state lives in `use-save-profile.ts`, as checkout's form does with `use-place-order.ts`): prefill; inline client errors with no fetch; Saving… state; "Profile updated." then auto-hide; the email-taken inline error; the generic error note; values kept on error (AC6-AC8, AC11, AC12)
- [x] T15 Implement `AccountDetailsForm.tsx` with `FormField`, to pass T14
- [x] T16 Write failing tests for `SignOutButton.tsx` (Signing out… and disabled, then POST to logout, then push to Home) and `AccountOverview.tsx` (section order, "0 saved", both empty states and links) (AC3, AC13)
- [x] T17 Implement `SignOutButton.tsx` and `AccountOverview.tsx` to pass T16
- [x] T18 Implement `AccountPage.tsx` (breadcrumb, hero with live name, tabs, panels, footer bar) and the `index.ts` barrel; test that the tab clicks switch panels and the welcome line updates after a save (AC2, AC4, AC8, AC14)
- [x] T19 Add `app/[locale]/account/page.tsx`: the `getSession` → `getMe` → `redirect(loginHref(ACCOUNT_HREF))` guard, force-dynamic and noindex metadata; test the redirect (AC1)
- [x] T20 Responsive pass against `visuals/my-account-notes.md` at 1280, 900 and 375 px: name row stacks at ≤480, 16px gutter, tabs `overflow-x-auto`, no page-level horizontal scroll (AC15). **Verified 2026-10-13** with headless Chromium at 1280/900/375 (overview, `#details`, `#orders`): no horizontal overflow at any width; name row `row` → `column` at 375; gutter 24px → 16px

## Verification

- [x] T21 Run the full frontend test suite plus lint and typecheck (testing skill / `/verify`). **2026-10-13:** frontend 83 files / 739 tests pass, `tsc` clean, `next lint` clean (one pre-existing font warning in the layout), `next build` compiles (`/account` shows ● like `/checkout`, from the layout's `generateStaticParams`; `force-dynamic` still gates per request). cms 16 files / 252 tests pass (21 new in `cms/tests/account/`), `tsc` clean
- [x] T22 Run QA against AC1-AC15 (qa skill) against a running cms + frontend, including the signed-out redirect round trip, a duplicate-email save, and the three viewport widths. Live browser checks already done 2026-10-13 (redirect → login with `?next` → back to `/account`; `#orders` scroll; `hashchange` → Details; UI save → welcome line updates, note hides after ~2.5s; Sign Out → `/` with the session cookie gone), **Run 2026-10-14:** PASS WITH NOTES, 15/15 AC — see qa-report.md

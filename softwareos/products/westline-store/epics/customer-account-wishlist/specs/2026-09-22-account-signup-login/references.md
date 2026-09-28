# References for Account Signup & Login

## Related Specs

_None._ (cross-epic/cross-product overlap search in Step 4 found no existing spec claiming any of this spec's covered paths — every `specs.index.yml` in the tree was still empty at shaping time.)

## Upstream Meeting

_None — no meeting write-ups exist yet._

## Similar Implementations

_None in-repo — `web/` is still create-next-app boilerplate and `cms/` has no content-types, so nothing existing to study._

## Visuals

- `login.html`, `signup.html` — static HTML/CSS mockups (see `visuals/`). Establish the WESTLINE design system (Poppins/Inter/JetBrains Mono, `--horizon` #155EEF accent), the shared `.auth-card` layout, and the exact field sets described in Technical Approach:
  - Login: email + password, "Forgot your password?" link (originally stubbed — see the 2026-09-23 change, now real), account-icon active state in nav.
  - Signup: first name + last name + email + password (8-char hint), no confirm-password field, no terms checkbox.
  - Both pages link to a "Seller Login" in the footer — out of scope for this spec (separate seller-dashboard epic).
- `forgot-password.html`, `reset-password.html` — static HTML/CSS mockups added 2026-09-23 for the forgot/reset password change (see `visuals/`). Both use the same `.auth-card` design system and an in-place "confirm state" swap (no navigation) rather than a redirect:
  - Forgot-password: email field, "Send Reset Link" → swaps to a "Check Your Email" state. The mockup's confirm state includes an "Open Reset Link" button that "simulates clicking the link you'd receive" (its own note: this is a static prototype with no real backend) — the real implementation does **not** carry this button over, since we don't expose the actual reset token client-side; the confirmation copy instead points at the `cms` server console (see Technical Approach's flagged email-delivery tradeoff).
  - Reset-password: new password + confirm password fields (8-char hint), "Reset Password" → swaps to a "Password Reset" success state linking back to sign in. The mockup assumes a working demo flow with no missing-code case; the real implementation adds a distinct "invalid or missing link" state (AC16) since a real `code` query param can genuinely be absent or wrong.

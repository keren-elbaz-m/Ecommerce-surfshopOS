# Checkout design notes

Source: `westline-site-desing/checkout.html`. The confirmation is the same page's `.is-confirmed` state. Where the spec decided differently from the design, the decision is marked **→ spec**.

## Page

- Title "Checkout — WESTLINE"; `h1.display` "Checkout". It is one page with no steps and no anchors.
- The design uses a minimal header (wordmark + cart badge) with no breadcrumb. **→ spec:** the regular SiteNav is kept.
- **Columns:**
  - Form column: 58%, white, with an inner max-width of 640px aligned right.
  - Summary column: 42%, `#f5f5f5` with `border-left: 1px solid #D6D6D6`. The summary inside is sticky at `top: 96px` with max-width 460px. **→ spec:** the summary uses the `bg-surface` token.
- **Breakpoints:**
  - ≤900px: the columns stack and the summary goes below with a top border.
  - ≤520px: `.co-row` stacks.
  - ≤480px: container padding becomes 16px.

## Form sections

Labels are Inter 12.5px/600, muted, not uppercase. Inputs have a 9px radius and a 1px border; the focus state is a Horizon border plus `0 0 0 3px rgba(21,94,239,.12)`. The design has no error states. **→ spec:** inline `text-danger` errors are added.

1. **Contact**
   - The design has an Email field (`you@email.com`) and a "Sign in" link on the right.
   - **→ spec:** the shopper is always signed in, so the section reads "Signed in as {email}".
2. **Delivery**
   - Fields:
     - Country/Region `<select>`
     - First name (`Jamie`) and Last name (`Rivera`) on one row
     - Address (`Street address`)
     - "Apartment, suite, etc. (optional)"
     - City, State and ZIP code on one row
     - Phone (`For delivery updates`, optional)
   - The design offers US and Canada, with State as free text. **→ spec:** US only, so Country is fixed text and State is a `<select>`.
3. **Shipping Method**
   - The design has one static option: "Standard Shipping", "Arrives in 5–7 business days", Free or $12.
   - **→ spec:** Standard + Express radio cards.
4. **Payment**
   - Note: "All transactions are simulated for this demo — no real payment is processed."
   - A pay-method box with a "Credit card" radio header, logos VISA / Mastercard / AMEX and a "+5" chip.
   - Card number (`1234 1234 1234 1234`), "Expiration date (MM/YY)", "Security code" (`CVC`), "Name on card" (`Jamie Rivera`).
   - **→ spec:** the "Use shipping address as billing address" checkbox is hidden.
   - CTA: "Place Order", full width, uppercase, Horizon. While pending: spinner + "Processing Payment…".
   - Fine print: "By placing your order, you agree to WESTLINE's Terms of Service and Privacy Policy. This is a demo storefront — no charge will occur."

## Order Summary sidebar

- Heading "Order Summary".
- **Each line:**
  - a 76×92 image with 9px radius and 1.5px border;
  - a dark circular badge showing the line index; **→ spec:** hidden;
  - the title and a meta line, with " · Qty N" added when qty is above 1;
  - the price in mono.
- Discount code + Apply. **→ spec:** hidden.
- Subtotal / Shipping / Total (bold, top border). There is no tax row.
- Note: "Taxes and shipping are demo placeholders. No real order is created."
- The design's constants (`FREE_SHIP_THRESHOLD = 900`, `FLAT_SHIP_FEE = 12`) are demo values. **→ spec:** `FREE_SHIPPING_THRESHOLD = 75` (the code's constant) with Standard at $12.

## Confirmation (`.is-confirmed`)

- Centred, max-width 640px.
- A 64px Horizon circle with a check.
- `h1` "Thank You!"
- "Your order has been placed successfully."
- "A confirmation has been sent to **{email}**."
- An order pill "Order #WL-XXXXXX" (mono 22px, surface background).
- A card with h3 "Order Summary": the line rows plus "Total Paid".
- Actions:
  - "View Order" (primary). It is a dead link in the design ("Order history is coming soon"). **→ spec:** hidden.
  - "Continue Shopping" (secondary, 2px Ink outline).

## Related designs

- **`profile.html` Order History:** accordion rows showing the number (`WL-104233`), "Placed Aug 22, 2026", the item count, a status pill (Delivered / Shipped / Processing) and the total. That screen belongs to customer-account-wishlist/order-history.
- **`seller-orders.html`:**
  - Columns: Order # / Customer / Date / Items / Total / Status.
  - Statuses: `processing | shipped | delivered`.
  - "Ship to" line.
  - **→ spec:** the enum adds `cancelled`.
- **Order number format:** everywhere it is `WL-` + 6 digits.

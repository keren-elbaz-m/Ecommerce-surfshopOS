# Cart & Checkout — Product Brief

> Epic: epic.md · Product: ../../mission.md

## Goals

storefront-catalog gets items into a client-side cart, but there's no persistent cart page, no way to check out, and no order confirmation — shoppers can't actually buy anything. Goal: take a cart full of items through checkout with mock payment to a confirmed order.

## User Stories

- As a shopper, I can view my cart so I can review what I'm about to buy.
- As a shopper, I can update quantities or remove items so my cart reflects what I actually want.
- As a shopper, I can check out with a mock payment so I can complete a purchase.
- As a shopper, I can see an order confirmation so I know my order went through.

## Success Criteria

- A shopper can open the cart, adjust quantities/remove items, proceed through checkout with mock payment, and land on an order confirmation page — with the order recorded in Strapi.

## Out of Scope

- Customer accounts/login (Customer Account epic)
- Wishlist (Wishlist epic)
- Surfboard Finder (Surfboard Finder epic)
- Seller dashboard / inventory management (Seller Dashboard epic)
- Real payment gateway integration

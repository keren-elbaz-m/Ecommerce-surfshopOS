# WESTLINE Ecosystem

> Customer: ../customer/overview.md
> Updated: 2026-09-22

## Map

```
westline
└── products
    └── westline-store — surf shop storefront + seller dashboard (repo: Ecommerce-surfshop, dirs: web/, cms/)
```

Single-product customer today — `cms` is westline-store's own backend, not a service shared across products. No shared services or other products discovered.

## Relationships

| Consumer | Depends on | Via | Notes |
|---|---|---|---|
| web | cms | REST (planned) | Not wired yet — no API client exists in `web/` today |

## Notes

Product Registry is generated separately at `ecosystem/products.index.yml` (rebuilt by `/refresh-indexes`).

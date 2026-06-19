# Pricing / NAV Issues

## Issue: Implement NAV calculator

Requirement IDs:

```txt
PRICE-001
PRICE-002
```

Acceptance criteria:

```txt
- uses reserve balances
- uses approved prices
- handles zero supply with initial NAV = 1
```

## Issue: Implement pricing source validation

Requirement IDs:

```txt
PRICE-003
PRICE-004
PRICE-005
PRICE-006
PRICE-007
PRICE-008
PRICE-009
PRICE-010
```

Acceptance criteria:

```txt
- supports source types
- checks enabled
- checks freshness
- checks stable depeg
- checks LST exchange rate
- rejects invalid StockToken pricing
```

## Issue: Implement pricing deviation checks

Requirement IDs:

```txt
PRICE-011
```

Acceptance criteria:

```txt
- computes deviation bps
- rejects deviation above threshold
```

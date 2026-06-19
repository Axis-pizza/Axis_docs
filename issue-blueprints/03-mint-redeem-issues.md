# Mint / Redeem Issues

## Issue: Implement mint allocation calculator

Requirement IDs:

```txt
MINT-003
MINT-004
MINT-005
```

Acceptance criteria:

```txt
- computes allocation by weight
- rejects allocation below 1 USDC
- rejects allocation above max_trade_usdc
```

## Issue: Implement Mint instruction

Requirement IDs:

```txt
MINT-001
MINT-002
MINT-006
MINT-007
MINT-008
MINT-009
MINT-010
MINT-011
MINT-012
MINT-013
MINT-014
```

Acceptance criteria:

```txt
- USDC input only
- active market only
- mint_enabled assets only
- approved routes only
- min_out enforced
- actual balance delta used
- actual added value used
- initial NAV handled
- all-or-nothing execution
```

## Issue: Implement redeem share calculator

Requirement IDs:

```txt
REDEEM-002
REDEEM-003
```

Acceptance criteria:

```txt
- calculates redeem share
- calculates pro-rata reserve amounts
```

## Issue: Implement Redeem instruction

Requirement IDs:

```txt
REDEEM-001
REDEEM-004
REDEEM-005
REDEEM-006
REDEEM-007
REDEEM-008
REDEEM-009
REDEEM-010
```

Acceptance criteria:

```txt
- validates DTF input mint
- redeem_enabled assets only
- approved routes only
- min_usdc_out enforced
- actual USDC delta used
- burns/escrows DTF
- all-or-nothing execution
```

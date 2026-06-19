# Swap CPI Adapter Issues

## Issue: Define generic venue adapter interface

Requirement IDs:

```txt
EXEC-014
```

Acceptance criteria:

```txt
- interface supports mint and redeem direction
- interface supports amount_in and min_out
- interface returns actual_received or lets Axis measure delta
```

## Issue: Implement Orca Whirlpool CPI spike

Requirement IDs:

```txt
EXEC-013
NFR-CU-001
```

Acceptance criteria:

```txt
- execute Whirlpool SwapV2 via CPI
- validate accounts
- measure compute units
- measure account count
- document Pinocchio/no_std gotchas
```

## Issue: Implement route validation

Requirement IDs:

```txt
EXEC-001
EXEC-002
EXEC-003
EXEC-004
EXEC-005
EXEC-011
```

Acceptance criteria:

```txt
- approved venue only
- approved route only
- input/output mint match
- pool id match
- direction match
- no arbitrary route graph
```

## Issue: Implement actual balance delta helper

Requirement IDs:

```txt
EXEC-008
MINT-009
REDEEM-008
```

Acceptance criteria:

```txt
- reads pre balance
- reads post balance
- computes delta
- validates delta direction
```

# DTF Market Issues

## Issue: Implement DTFMarket account

Requirement IDs:

```txt
DTF-010
DTF-011
DTF-012
```

Acceptance criteria:

```txt
- stores creator
- stores DTF mint
- stores asset count
- stores status
- stores reserve authority
```

## Issue: Implement CreateMarket validation

Requirement IDs:

```txt
DTF-001
DTF-002
DTF-003
DTF-004
DTF-005
DTF-006
DTF-007
DTF-008
DTF-009
```

Acceptance criteria:

```txt
- reject 1 asset DTF
- reject >5 assets
- reject duplicate assets
- reject invalid total weights
- reject weight below 100 bps
- reject weight above asset max
- reject unregistered asset
- reject creation_disabled asset
```

## Issue: Implement reserve account validation

Requirement IDs:

```txt
DTF-010
NFR-SEC-004
```

Acceptance criteria:

```txt
- reserve account mint matches asset mint
- reserve authority is Axis-controlled
- invalid reserve account fails
```

## Issue: Implement market fee state

Requirement IDs:

```txt
DTF-013
DTF-014
DTF-015
DTF-016
DTF-017
FEE-002
FEE-003
FEE-004
FEE-005
FEE-016
```

Acceptance criteria:

```txt
- stores creator and creator_fee_destination
- snapshots fee config (mint/redeem fee bps, creator/protocol shares) from protocol config at creation
- fee bps are not creator-customizable
- market fee config immutable after creation
- tracks accrued_creator_fee_usdc and accrued_protocol_fee_usdc
- fee custody account is separate from reserve token accounts
- fee vault balance excluded from reserve value and NAV
```

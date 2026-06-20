# Testing / Security Issues

## Issue: Implement market creation tests

Requirement IDs:

```txt
TEST-DTF-001 through TEST-DTF-010
```

Acceptance criteria:

```txt
- all composition edge cases covered
```

## Issue: Implement mint tests

Requirement IDs:

```txt
TEST-MINT-001 through TEST-MINT-010
```

Acceptance criteria:

```txt
- allocation, policy, route, pricing, balance delta tests covered
```

## Issue: Implement redeem tests

Requirement IDs:

```txt
TEST-REDEEM-001 through TEST-REDEEM-008
```

Acceptance criteria:

```txt
- redeem share and unwind tests covered
```

## Issue: Implement emergency tests

Requirement IDs:

```txt
TEST-ADMIN-001 through TEST-ADMIN-005
```

Acceptance criteria:

```txt
- exit-only mode works
- unauthorized changes fail
```

## Issue: Implement CPI spike tests

Requirement IDs:

```txt
TEST-CPI-001 through TEST-CPI-007
```

Acceptance criteria:

```txt
- Orca CPI executable
- compute and account limits documented
- multi-swap feasibility tested
```

## Issue: Implement fee tests

Requirement IDs:

```txt
TEST-FEE-001 through TEST-FEE-017
FEE-018
```

Acceptance criteria:

```txt
- mint fee calculation and creator/protocol split covered
- net composition allocation covered
- redeem zero-fee behavior covered
- fee config validation and caps covered
- immutable market fee config covered
- claim and double-claim/authorization cases covered
- accrued fees excluded from NAV and reserve backing covered
- fee custody / reserve separation covered
```

## Issue: Implement pre-mainnet validation suite

Requirement IDs:

```txt
PMV-001 through PMV-013
```

Acceptance criteria:

```txt
- local / LiteSVM / local validator / fork-based tests cover the core lifecycle
- production venue CPI integration path validated
- app contract integration smoke tests pass
- guarded mainnet launch controls configured
- public Devnet is not a required readiness gate
```

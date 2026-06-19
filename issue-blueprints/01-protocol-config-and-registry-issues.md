# Protocol Config & Registry Issues

## Issue: Implement ProtocolConfig account

Requirement IDs:

```txt
ADMIN-001
NFR-UPG-001
```

Acceptance criteria:

```txt
- stores authority
- stores pause authority
- stores registry authorities
- stores default USDC mint
- stores protocol treasury
- unauthorized initialization/update fails
```

## Issue: Implement AssetRegistry

Requirement IDs:

```txt
ASSET-001
ASSET-002
DTF-007
```

Acceptance criteria:

```txt
- register asset mint
- store category
- store decimals
- store manual review flag
- reject duplicate registration
```

## Issue: Implement AssetExecutionPolicy

Requirement IDs:

```txt
POLICY-002
POLICY-003
POLICY-004
POLICY-005
POLICY-010
```

Acceptance criteria:

```txt
- policy exists per asset
- supports flags
- supports max_trade_usdc
- supports max_weight_bps
- supports max_price_impact_bps
- supports pricing requirement
```

## Issue: Implement ApprovedRouteRegistry

Requirement IDs:

```txt
EXEC-001
EXEC-002
EXEC-003
EXEC-004
```

Acceptance criteria:

```txt
- route can be registered by authorized role
- route can be disabled
- route validates input/output mint
- route validates venue and pool
```

## Issue: Implement PricingSourceRegistry

Requirement IDs:

```txt
PRICE-003
PRICE-004
PRICE-005
```

Acceptance criteria:

```txt
- pricing source can be registered
- pricing source can be disabled
- source type stored
- staleness/deviation params stored
```

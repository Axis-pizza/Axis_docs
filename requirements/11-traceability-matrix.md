# Traceability Matrix

This file maps requirement areas to implementation tasks.

| Requirement Area | Requirement IDs | Implementation Surface | Issue Blueprint |
|---|---:|---|---|
| Protocol config | AXIS-CORE, ADMIN | ProtocolConfig account | `01-protocol-config-and-registry-issues.md` |
| Asset registry | ASSET, POLICY | AssetRegistry, AssetExecutionPolicy | `01-protocol-config-and-registry-issues.md` |
| DTF market | DTF | DTFMarket, MarketAssetWeight | `02-dtf-market-issues.md` |
| Mint | MINT | Mint instruction | `03-mint-redeem-issues.md` |
| Redeem | REDEEM | Redeem instruction | `03-mint-redeem-issues.md` |
| Swap CPI | EXEC | ApprovedRoute, VenueAdapter | `04-swap-cpi-adapter-issues.md` |
| Pricing / NAV | PRICE | PricingSource, NAV calculator | `05-pricing-nav-issues.md` |
| Admin / safety | ADMIN | SetPolicy, Pause, Route updates | `06-testing-security-issues.md` |
| Testing | TEST, NFR | Unit/integration/property tests | `06-testing-security-issues.md` |

## Required Issue Metadata

Each issue should include:

```txt
Requirement IDs:
Design docs:
Acceptance criteria:
Out of scope:
Test cases:
Dependencies:
```

## Example

```txt
Title:
Implement DTF market creation validation

Requirement IDs:
DTF-001, DTF-002, DTF-003, DTF-004, DTF-005, DTF-006, DTF-007, DTF-008

Acceptance criteria:
- 1 asset DTF fails
- 2 asset DTF succeeds
- 6 asset DTF fails
- duplicate assets fail
- invalid weight sum fails
- weight below 100 bps fails
- weight above asset max fails
- creation_enabled=false fails
```

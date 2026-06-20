# Traceability Matrix

This file maps requirement areas to implementation tasks.

| Requirement Area | Requirement IDs | Implementation Surface | Issue Blueprint |
|---|---:|---|---|
| Protocol config | AXIS-CORE, ADMIN | ProtocolConfig account | `01-protocol-config-and-registry-issues.md` |
| Asset registry | ASSET, POLICY | AssetRegistry, AssetExecutionPolicy | `01-protocol-config-and-registry-issues.md` |
| DTF market | DTF (incl. DTF-013..017 fee state) | DTFMarket, MarketAssetWeight, MarketFeeState, fee vault | `02-dtf-market-issues.md` |
| Mint | MINT (incl. MINT-003..018 net fee accounting) | Mint instruction, mint fee deduction/accrual | `03-mint-redeem-issues.md` |
| Redeem | REDEEM (incl. REDEEM-011..018 zero-fee) | Redeem instruction | `03-mint-redeem-issues.md` |
| Swap CPI | EXEC (incl. EXEC-015..017 two-venue readiness) | ApprovedRoute, VenueAdapter, Orca Whirlpool + Raydium CPMM paths | `04-swap-cpi-adapter-issues.md` |
| Pricing / NAV | PRICE | PricingSource, NAV calculator | `05-pricing-nav-issues.md` |
| Fee model | FEE | ProtocolFeeConfig, MarketFeeState, fee vault, fee claim instructions | TODO: fee issue blueprint not yet created |
| Pre-mainnet validation | PMV | Local / LiteSVM / fork tests, two-venue CPI tests, guarded launch controls | `06-testing-security-issues.md` |
| Admin / safety | ADMIN | SetPolicy, Pause, Route updates, protocol fee config (capped) | `06-testing-security-issues.md` |
| Testing | TEST, NFR | Unit/integration/property tests | `06-testing-security-issues.md` |

TODO: requirement areas referenced in `00-requirements-overview.md §5` (VENUE-*, APP-*, BD-*, MAINNET-*) do not yet have dedicated requirement docs or matrix rows. VENUE/MAINNET concerns currently live inside `05` (EXEC-*) and `12` (PMV-*). Add dedicated rows if/when standalone docs are created — do not invent IDs.

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

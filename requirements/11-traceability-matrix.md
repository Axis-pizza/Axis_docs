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
| Secondary market | SECONDARY-001..008 | Axis-operated discovery surface, external-pool indexing and labels | TODO: secondary-market issue blueprint |
| Axis Auction Program | AUCTION-001..019 | Proposed Auction Program, Orca JIT spike, ClearCorrection activation gates | TODO: auction-program issue blueprint |

TODO: requirement areas referenced in `00-requirements-overview.md §5` (VENUE-*, APP-*, BD-*, MAINNET-*) do not yet have dedicated requirement docs or matrix rows. VENUE/MAINNET concerns currently live inside `05` (EXEC-*) and `12` (PMV-*). Add dedicated rows if/when standalone docs are created — do not invent IDs.

## Secondary Market and Auction Traceability

Source documents:

- [Requirements Overview](00-requirements-overview.md)
- [Definitions & Decision Log](01-definitions-and-decision-log.md)
- [Auction and LVR Design Research](17-auction-and-lvr-design-research.md)
- [Secondary Market and ClearCorrection Requirements](18-secondary-market-and-clear-correction-requirements.md)
- [Swap CPI Execution Requirements](05-swap-cpi-execution-requirements.md)
- [Production Venue Integration Requirements](14-production-venue-integration-requirements.md)
- [Route Builder Backend API Requirements](16-route-builder-backend-api-requirements.md)
- [Pre-Mainnet Validation Requirements](12-pre-mainnet-validation-requirements.md)

| Requirement ID | Implementation surface | Design and validation evidence |
|---|---|---|
| SECONDARY-001 | Market discovery, DTF mint visibility, secondary-liquidity status | `18` §3; `00` Phase 4; `12` PMV-015 |
| SECONDARY-002 | Partner/sponsor campaign metadata and canonical market/share URL | `18` §3; `00` Phase 4; `12` PMV-015 |
| SECONDARY-003 | External DTF/USDC pool indexing and verification | `18` §3; `16` RBA-037; `12` PMV-015 |
| SECONDARY-004 | External-liquidity labels and no false LVR claim | `01` §§22–23; `16` RBA-038; `12` PMV-015 |
| SECONDARY-005 | Permissionless external-pool policy and selective indexing | `18` §3; `01` §22 |
| SECONDARY-006 | External price/NAV separation | `18` §4; `06-pricing-nav-requirements.md`; `12` PMV-015 |
| SECONDARY-007 | External pools have no Axis-native LVR claim | `17` §2; `18` §4; `16` RBA-038 |
| SECONDARY-008 | External-pool failures cannot affect Axis Core correctness | `18` §4; `05` §§1–2; `12` PMV-008 |
| AUCTION-001 | DTF-market architectural compatibility | `01` §25; `17` §2; `18` §5 |
| AUCTION-002 | Explicit, per-market native-liquidity activation status | `18` §5; `16` RBA-039; `12` PMV-017 |
| AUCTION-003 | Orca Whirlpool as first Role B settlement candidate | `01` §24; `05` EXEC-021; `14` VENUE-024 |
| AUCTION-004 | Axis-controlled native correction authorization | `17` §§3–4; `18` §5; `14` VENUE-024 |
| AUCTION-005 | Axis Core remains primary accounting authority | `01` §26; `18` §6; existing Core mint/redeem tests |
| AUCTION-006 | Separate Auction Program and ClearCorrection boundary | `01` §26; `17` §2; `18` §6; proposed instruction/account specs |
| AUCTION-007 | Dedicated AuctionRevenueVault accounting | `01` §28; `18` §6; `12` PMV-016/017 |
| AUCTION-008 | Winner-authorized, bounded ClearCorrection validation | `18` §7; `17` §4; `12` PMV-016 |
| AUCTION-009 | Single-transaction ClearCorrection target and rollback behavior | `01` §27; `05` EXEC-021; `12` PMV-016/017 |
| AUCTION-010 | Constrained Jito fallback | `01` §27; `17` §2; `14` VENUE-024; `12` PMV-016 |
| AUCTION-011 | Core validations retained when correction involves mint/redeem | `18` §7; `05` §§1–2; `16` RBA-040 |
| AUCTION-012 | Per-market activation gates | `01` §25; `18` §8; `12` PMV-017 |
| AUCTION-013 | Inactive markets remain valid Axis DTFs | `18` §8; `16` RBA-038; `12` PMV-015 |
| AUCTION-014 | Reversible native-liquidity disablement | `18` §8; `12` PMV-017; admin/safety validation |
| AUCTION-015 | Orca JIT settlement technical spike | `17` §7; `05` EXEC-021; `14` VENUE-024; `12` PMV-016 |
| AUCTION-016 | Atomicity and resource measurement spike | `17` §7; `05` §9.6; `12` PMV-016 |
| AUCTION-017 | Pricing and route-support spike | `17` §6; `18` §9; `05` §§1–2; `12` PMV-016 |
| AUCTION-018 | Winner/interception/replay safety spike | `17` §6; `18` §9; `12` PMV-016 |
| AUCTION-019 | Auction revenue separation spike | `17` §7; `18` §9; `12` PMV-016/017 |

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

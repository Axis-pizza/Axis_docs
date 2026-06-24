# Traceability Matrix

This file maps requirement areas to implementation tasks.

| Requirement Area                       |          Requirement IDs | Implementation Surface                                                                   | Issue Blueprint                               |
| -------------------------------------- | -----------------------: | ---------------------------------------------------------------------------------------- | --------------------------------------------- |
| Protocol config                        |         AXIS-CORE, ADMIN | ProtocolConfig account, authority config, protocol-level settings                        | `01-protocol-config-and-registry-issues.md`   |
| Asset registry                         |            ASSET, POLICY | AssetRegistry, AssetExecutionPolicy, asset enablement flags                              | `01-protocol-config-and-registry-issues.md`   |
| DTF market                             |                      DTF | DTFMarket, MarketAssetWeight, reserve vaults, DTF mint                                   | `02-dtf-market-issues.md`                     |
| Mint                                   |                     MINT | Mint instruction, USDC input, reserve composition, DTF minting, balance delta accounting | `03-mint-redeem-issues.md`                    |
| Redeem                                 |                   REDEEM | Redeem instruction, DTF burn, reserve unwind, USDC output, min_usdc_out                  | `03-mint-redeem-issues.md`                    |
| Swap CPI execution                     |                     EXEC | ApprovedRoute, VenueAdapter, controlled adapter, production venue adapters               | `04-swap-cpi-adapter-issues.md`               |
| Pricing / NAV                          |                    PRICE | PricingSource, NAV calculator, reserve value calculation                                 | `05-pricing-nav-issues.md`                    |
| Fee model                              |                      FEE | ProtocolFeeConfig, MarketFeeState, fee vault, creator/protocol claim instructions        | `07-fee-accounting-issues.md`                 |
| Pre-mainnet validation                 |                      PMV | Local tests, LiteSVM tests, fork tests, venue CPI tests, guarded launch controls         | `06-testing-security-issues.md`               |
| Admin / safety                         |                    ADMIN | SetPolicy, Pause, route updates, asset disable, emergency controls                       | `06-testing-security-issues.md`               |
| Testing / non-functional requirements  |                TEST, NFR | Unit tests, integration tests, failure tests, compute/account measurements               | `06-testing-security-issues.md`               |
| Production venue integration           |              VENUE, EXEC | Orca Whirlpool, Raydium CPMM fallback, venue-specific validation evidence                | `04-swap-cpi-adapter-issues.md`               |
| App contract interface                 |                      APP | App/client read model, transaction construction inputs, contract state integration       | `08-app-contract-interface-issues.md`         |
| Route builder backend API              |         APP, EXEC, PRICE | Route plan API, quote freshness, ApprovedRoute references, account assembly              | `09-route-builder-api-issues.md`              |
| Mainnet launch gate                    | MAINNET, PMV, ADMIN, NFR | Launch checklist, deployment traceability, guarded controls, accepted risk log           | `10-mainnet-launch-gate-issues.md`            |
| Partner demo / BD readiness            |       BD, PMV, SECONDARY | Contract evidence, partner demo flows, canonical market URL, integration narrative       | `11-partner-demo-readiness-issues.md`         |
| Secondary market                       |       SECONDARY-001..008 | Axis-operated discovery surface, external-pool indexing, external-liquidity labels       | `12-secondary-market-surface-issues.md`       |
| Axis Auction Program / ClearCorrection |         AUCTION-001..019 | Proposed Auction Program, Orca JIT spike, ClearCorrection activation gates               | `13-auction-clear-correction-spike-issues.md` |

Notes:

* `07-fee-accounting-issues.md` is a P0 issue blueprint because creator/protocol fee accounting is required from Axis v1 launch.
* `08-app-contract-interface-issues.md`, `09-route-builder-api-issues.md`, and `10-mainnet-launch-gate-issues.md` may be created after the first Axis Core scaffold issues, but they should remain traceable from this matrix.
* `12-secondary-market-surface-issues.md` is required for launch-day product readiness, but it is not part of Axis Core reserve custody or mint/redeem accounting.
* `13-auction-clear-correction-spike-issues.md` is a deferred / conditional activation blueprint. It is not a blocker for Axis Core P0 implementation or the minimum August launch gate unless Axis explicitly activates Axis-controlled JIT liquidity for a market.

## Deferred and Conditional Scope Notes

Axis Core P0 implementation should focus on reserve-backed DTF creation, mint, redeem, accounting, fees, policy validation, route validation, and testability.

Secondary-market surface requirements are launch-day product requirements, but external DTF/USDC pools are not Axis Core reserve custody, NAV, or mint/redeem accounting.

Axis Auction Program and ClearCorrection requirements are architectural compatibility and technical-spike requirements unless a market is explicitly activated for Axis-controlled JIT liquidity.

Therefore:

```txt
Axis Core P0:
- create / mint / redeem
- reserves / NAV / accounting
- fees
- asset policy
- ApprovedRoute validation
- controlled adapter tests
- production venue validation path

Launch-day product readiness:
- secondary-market surface
- external-liquidity labels
- partner-facing market metadata

Deferred / conditional activation:
- Axis Auction Program
- ClearCorrection
- Axis-controlled JIT liquidity
```


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

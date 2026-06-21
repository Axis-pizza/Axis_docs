# Secondary Market and ClearCorrection Requirements

## 1. Scope

This document defines implementation-facing requirements for Axis v1 secondary-market presentation, external DTF/USDC pools, Axis-controlled JIT liquidity, and ClearCorrection.

It preserves the primary Axis Core model:

```txt
Axis Core owns DTF creation, mint, redeem, reserves, NAV, and accounting.
Mint is USDC-in and redeem is USDC-out.
Mint/redeem execution remains route-builder prepared and on-chain validated.
```

The design rationale and remaining technical research are in `17-auction-and-lvr-design-research.md`.

## 2. Launch Scope and Gating

The launch-day requirement is a secondary-market surface. Production-grade Axis-controlled JIT liquidity, ClearCorrection, and the Axis Auction Program are not August launch blockers.

All DTF markets must be architecturally compatible with later native-liquidity activation, but a market is not auction/JIT enabled until it passes every applicable activation gate in this document.

## 3. Secondary-Market Surface Requirements

### SECONDARY-001: DTFs must be discoverable as secondary-market assets

Axis must provide an app or equivalent Axis-operated surface on which a created DTF market can be discovered and identified for secondary-market use.

The surface must provide, directly or through a linked market detail view:

```txt
- DTF market identity and token mint
- current market status, including mint/redeem availability
- reserve/NAV context that is explicitly distinguished from a secondary-pool price
- eligible secondary-liquidity references, when known
- a status label that distinguishes external liquidity from Axis-native auction-enabled liquidity
- a canonical market URL or share URL and stable partner-facing market metadata
```

### SECONDARY-002: Partner and sponsor distribution must be supported

The market identity and token information exposed by the secondary-market surface, including the canonical market URL or share URL and partner-facing metadata, must be usable for partner and sponsor campaign integrations. This is a distribution requirement; it does not grant a partner control over Axis Core, reserves, NAV, or auction settlement.

### SECONDARY-003: External pools may be indexed or displayed

Axis may index or display a public third-party DTF/USDC pool after validating enough metadata to identify the DTF mint, USDC mint, venue, and pool address correctly.

Indexing or display does not:

```txt
- approve the pool as Axis-native liquidity
- make the pool part of DTF reserves or NAV
- authorize the pool for mint/redeem CPI execution
- guarantee liquidity, execution quality, price accuracy, or LVR protection
```

### SECONDARY-004: External liquidity must be labelled precisely

Every user-facing or partner-facing representation of a public third-party pool must identify it as external liquidity. Axis must not state or imply that the pool is LVR-mitigated, auction-enabled, reserve-backed by that pool, or controlled by Axis solely because it trades an Axis-issued DTF.

### SECONDARY-005: Public pool creation remains permissionless to Axis

Axis must not require an Axis Core permission or registry entry for a third party to create a public DTF/USDC pool on an external venue. Axis indexing eligibility and UI display policy may be selective, but do not alter the pool's external status.

## 4. External-Pool Policy

### SECONDARY-006: External pool prices are not NAV

An external DTF/USDC pool price must not replace reserve NAV, Pricing Source Registry inputs, or Axis Core pricing validation. The app may display market and NAV values together only when their different sources and meanings are clear.

### SECONDARY-007: External pools have no native LVR claim

Axis-native LVR mitigation applies only to a market's activated Axis-controlled auction/JIT settlement path. No external public pool may receive an Axis LVR-mitigation claim, including a public pool on Orca or Raydium.

### SECONDARY-008: External pool failure must not affect Axis Core correctness

An unavailable, manipulated, drained, or stale external pool must not change Axis Core reserve accounting, DTF supply accounting, NAV, mint/redeem eligibility, or primary-flow route validity.

## 5. Native-Liquidity Architecture and Venue Direction

### AUCTION-001: All DTF markets must remain architecturally compatible

DTF market architecture must not permanently preclude later connection to Axis-controlled JIT liquidity and NAV Correction Auctions. Compatibility does not require a market to have an active pool, auction account, or JIT position at creation.

### AUCTION-002: Activation must be explicit and per market

An Axis-native liquidity configuration must be explicitly activated for its DTF market. The absence of an active configuration must be observable by the app and indexer and must result in no Axis-native LVR-mitigation claim.

### AUCTION-003: Orca Whirlpool is the first native settlement candidate

The first technical-spike and implementation candidate for Axis-controlled JIT liquidity is Orca Whirlpool.

This requirement does not replace the established primary-flow venue policy: Orca Whirlpool remains the first mint/redeem venue candidate and Raydium CPMM remains an important fallback. Raydium may also be external DTF/USDC liquidity, but it is not the first Axis-controlled JIT settlement target.

### AUCTION-004: Axis must control the native correction path

For a market to be described as Axis-native auction/JIT liquidity, the Axis Auction Program must control the correction authorization and settlement preconditions. A venue pool alone, including a public pool on Orca, is insufficient.

## 6. Axis Core and Auction Program Boundary

### AUCTION-005: Axis Core remains the primary accounting authority

Axis Core remains solely responsible for DTF market creation, mint, redeem, reserve custody, reserve accounting, NAV, pricing-source validation, and mint/redeem fee accounting.

### AUCTION-006: ClearCorrection belongs to the Axis Auction Program

The Axis Auction Program must own the ClearCorrection and NAV Correction Auction instruction logic, including auction lifecycle, winner authorization, correction-bound validation, settlement coordination, auction-payment settlement, and event emission.

The Auction Program must not:

```txt
- hold or account for DTF reserve backing as auction liquidity
- mutate DTF NAV as an auction result
- treat auction revenue as DTF reserve value
- bypass Axis Core validation when a correction strategy invokes mint or redeem
```

The route builder may prepare advisory venue, account, and transaction-plan inputs under its existing boundary. It cannot grant a correction right, determine the auction winner, or substitute for on-chain Auction Program or Axis Core validation.

### AUCTION-007: Auction revenue must use dedicated accounting

Auction payments and other auction revenue must be accounted for separately from DTF reserves and mint/redeem fees. The implementation should use an `AuctionRevenueVault` or an equivalent dedicated account model.

The exact vault layout, claim mechanics, denomination, recipient set, and revenue split are configurable/TBD. Auction revenue must be excluded from reserve value and NAV in every case.

## 7. ClearCorrection Settlement Requirements

### AUCTION-008: ClearCorrection must be winner-authorized and bounded

ClearCorrection must verify, before any successful settlement:

```txt
- the auction exists, is active, and is not expired
- the supplied DTF market and native-liquidity configuration match the auction
- the caller is the designated winner and cannot replay an already-used right
- pricing/NAV inputs meet the configured freshness and validation rules
- the correction direction and outcome meet configured bounds
- the auction payment is routed to dedicated auction-revenue accounting
```

Exact bounds, thresholds, maximum correction size, bid type, and auction duration are configurable/TBD.

### AUCTION-009: Single-transaction settlement is preferred

The production target for ClearCorrection is a single transaction containing the required authorization, validation, native-liquidity settlement, auction-payment settlement, and state updates.

If any required component fails, the transaction must not leave a partially successful correction, a successful auction-payment settlement without correction, or an Axis Core reserve-accounting change attributable to the failed correction.

### AUCTION-010: Jito is a constrained fallback only

Jito bundles may be used only if the single-transaction path has been shown infeasible and a technical spike demonstrates:

```txt
- ordered atomic execution across the complete correction sequence
- no non-winner interception opportunity between the ordered steps
- safe failure behavior equivalent to the single-transaction requirement
```

Jito bundle use is not, by itself, proof that these requirements are met.

### AUCTION-011: Correction interactions with Axis Core must preserve Core validation

When a correction strategy economically uses mint or redeem, the relevant Axis Core instruction path must retain its normal ApprovedRoute, pricing, min-out, balance-delta, reserve, and fee validations. The Auction Program may coordinate authorized settlement but cannot replace Axis Core as the accounting authority.

## 8. Per-Market Activation Gates

### AUCTION-012: All activation gates are mandatory

Axis-native auction/JIT liquidity may be enabled for a market only after evidence confirms all of the following:

```txt
- an eligible settlement pool/configuration is available
- required NAV and pricing inputs meet freshness and validation requirements
- the required route and venue-account support is available
- the Axis Auction Program supports the market and settlement configuration
- the complete path fits validated account and compute limits
- safety validation covers authorization, expiry, replay, stale inputs, incorrect pool inputs, failure rollback, and revenue separation
```

The activation result must be market-specific. Passing a technical spike for one market or pool configuration does not automatically enable another market.

### AUCTION-013: Inactive markets remain valid Axis markets

A market that has not passed the activation gates remains a valid reserve-backed Axis DTF if it satisfies the existing Core requirements. It may be shown in the secondary-market surface and may have external public liquidity, but must not be labelled as Axis-native LVR-mitigated liquidity.

### AUCTION-014: Native-liquidity activation must be reversible

The design must support safely disabling a market's native auction/JIT configuration when its pool, pricing, route, compute, or safety assumptions are no longer satisfied. Disabling native liquidity must not compromise Axis Core mint/redeem accounting or reserve exits.

## 9. Technical-Spike Requirements

### AUCTION-015: Orca JIT settlement spike

The Orca Whirlpool spike must demonstrate or reject the feasibility of Axis-controlled JIT liquidity for a bounded correction. It must document required authorities, account classes, Whirlpool operations, ordering assumptions, and failure behavior.

### AUCTION-016: Atomicity and resource spike

The spike must attempt the complete one-transaction ClearCorrection path and record account count, compute consumption, transaction size constraints, and rollback behavior. It must identify the supported DTF market classes, if any, without assuming that every 2-to-5 asset market is feasible.

### AUCTION-017: Pricing and route-support spike

The spike must validate fresh NAV/pricing input handling and the required route/venue-account support. It must include stale, missing, inconsistent, and incorrect-input failure cases.

### AUCTION-018: Authorization and interception spike

The spike must test wrong winner, expired auction, replay, wrong market, wrong pool, copied transaction inputs, and attempted non-winner settlement. If Jito is proposed, the same spike must provide evidence for the constrained fallback requirements in AUCTION-010.

### AUCTION-019: Revenue-separation spike

The spike must prove that successful auction revenue reaches only dedicated auction accounting and is excluded from DTF reserves, NAV, and mint/redeem fee accounting. It must also test failure paths so no successful payment is recorded for a failed correction.

## 10. Deferred Configuration

The following must remain configurable/TBD until technical-spike evidence supports concrete values:

```txt
- correction/deviation threshold
- maximum correction size
- JIT liquidity range, inventory, and lifecycle parameters
- auction duration, bid format, and winner-selection method
- auction revenue recipients and fee split
- initial market subset eligible for activation
```

No default values for these parameters are implied by this document.

## 11. Acceptance Summary

The launch-day secondary-market requirement is complete when DTFs are discoverable and accurately presented for secondary use, including clearly labelled external pool references where available.

A market-specific Axis-native LVR-mitigation claim is allowed only after the relevant native-liquidity implementation has passed AUCTION-012 and the supporting technical spikes. Until then, public DTF/USDC pools remain external liquidity.

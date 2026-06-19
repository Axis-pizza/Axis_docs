# Axis v1 Open Design Spec

## 1. Overview

Axis v1 is an open DTF protocol on Solana.

A **DTF**, or **Dex Traded Fund**, is a tradable position token backed by a basket of underlying reserve assets. Users can create DTFs from a curated asset universe, and other users can mint or redeem those DTFs with USDC.

Axis v1 is designed as an open protocol. It does **not** rely on market-level TVL caps. Instead, risk is controlled through asset-level execution policies, approved routing, pricing checks, and per-transaction limits.

## 2. Core Design Principles

### 2.1 Axis First

Axis must be able to create, mint, redeem, and account for DTFs independently.

External routers such as Titan may later route into Axis DTF markets, but they are not part of Axis Core.

```txt
Axis Core = DTF issuance, reserve accounting, mint, redeem, execution policy
Titan     = external routing / distribution / venue integration layer
```

### 2.2 Actual Reserve Backing

DTFs are backed by actual underlying reserve assets.

Axis v1 is **not** a synthetic USDC-reserve wrapper. When a user mints a DTF, the Axis program composes the underlying assets. When a user redeems, Axis unwinds the underlying assets back to USDC.

### 2.3 Controlled CPI Execution

Axis v1 uses controlled CPI execution.

Route discovery and account assembly may happen off-chain, but the Axis program executes swaps only through approved venue adapters.

The program does not blindly trust off-chain quotes. It verifies:

- approved assets
- approved venues
- route constraints
- `min_out`
- price impact
- pricing deviation
- actual balance deltas

### 2.4 Open Version

Axis v1 Open Version does not impose market-level TVL caps.

Risk is controlled through:

- asset-level max weight
- asset-level max trade size per transaction
- max price impact
- `min_out` checks
- approved route requirements
- pricing deviation checks
- actual balance delta checks
- asset-level enable / disable flags

## 3. Key Design Decisions

| Area | Decision |
|---|---|
| Protocol mode | Open version |
| Market-level TVL cap | None |
| Backing model | Actual reserve assets |
| Mint input | USDC |
| Redeem output | USDC |
| Asset count per DTF | Minimum 2, maximum 5 |
| One-asset DTF | Not allowed |
| Execution | Controlled CPI through approved venue adapters |
| First CPI spike candidate | Orca Whirlpool |
| Pricing model | Pricing Source Registry, not pure Oracle Registry |
| NAV accounting | Actual reserve balances × approved prices |
| Asset universe | Curated 500 assets |
| Titan role | External routing / distribution layer, not Axis Core |

## 4. Current Open Areas

```txt
1. Fee Model
2. Rebalance Scope
3. Titan Integration Boundary
4. Exact pricing source implementation
5. First approved production CPI venue after Orca spike
6. Launch-readiness classification for the 500-asset universe
```

# Swap CPI Execution

## 1. Goal

Axis v1 needs to compose and unwind up to 5 underlying assets for each DTF mint/redeem.

The protocol should not become a general-purpose DEX aggregator. It should be a controlled execution engine for DTF composition and redemption.

## 2. Core Principle

```txt
Route discovery / quote / account assembly = off-chain
Execution / verification / accounting       = on-chain
```

## 3. Approved Venue Adapter Model

```txt
Axis Core
  -> Approved Venue Adapter
    -> Orca Whirlpool
    -> Raydium CPMM
    -> PumpSwap
    -> Raydium CLMM
    -> Meteora DLMM
```

## 4. First Spike Target

Orca Whirlpool is the first CPI spike candidate because Axis has direct access to Orca team members for technical guidance.

However, the architecture must remain adapter-based. Orca is the first target, not the only execution venue.

## 5. Venue Research Summary

### Orca Whirlpool

Use case:

- stable
- LST
- major assets
- high-confidence routes

Open questions:

- exact compute cost per swap
- account count with multiple swaps
- tick array selection
- Pinocchio / no_std gotchas
- Token-2022 / transfer hook behavior

### Raydium CPMM

Use case:

- broad Solana token coverage
- simpler constant-product routes
- potential first non-Orca adapter

### PumpSwap

Use case:

- meme long-tail
- Pump graduation assets

Open questions:

- SOL quote path handling
- USDC -> SOL -> meme route complexity
- max 5 asset transaction feasibility

### Raydium CLMM

Use case:

- deeper liquidity / concentrated liquidity

Open questions:

- tick array complexity
- account count
- compute cost

### Meteora DLMM

Use case:

- volatile assets
- DLMM liquidity

Open questions:

- CPI spike required
- bin array / bitmap extension complexity
- account assembly complexity

## 6. Jupiter Role

Jupiter should be used for:

- route discovery
- asset coverage research
- price impact estimation
- venue discovery
- candidate route generation

Jupiter should not be the Axis Core execution layer for v1.

Important distinction:

```txt
Jupiter route_available = can likely swap through Jupiter
Axis approved CPI route = Axis program can safely execute via approved adapter
```

## 7. Swap Route Verification

Each swap must verify:

```txt
- asset is part of DTF market
- asset is enabled for mint/redeem
- venue is approved
- route is approved
- input mint matches expected mint
- output mint matches expected mint
- pool matches approved pool
- amount is within max_trade_usdc
- expected price impact is within max_price_impact_bps
- min_out is provided
- actual_received >= min_out
- actual balance delta matches expected direction
```

## 8. Route Complexity Limits

```txt
- no split routing
- one route per asset per execution
- prefer direct USDC <-> asset routes
- allow SOL intermediate only if explicitly supported later
- no arbitrary route graph execution inside Axis Core
```

## 9. Pending Orca Questions

Initial questions to confirm with Orca:

```txt
1. Is Whirlpool SwapV2 via CPI realistic for DTF mint/redeem execution?
2. Are there major gotchas for Pinocchio / no_std instead of Anchor?
3. Are multiple Whirlpool swap CPIs in one transaction realistic for up to 5 assets?
```

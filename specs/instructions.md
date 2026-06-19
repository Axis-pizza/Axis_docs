# Instruction Spec

This document lists conceptual Axis v1 instructions. Exact names and accounts may change during implementation.

## CreateMarket

Creates a new DTF market.

Inputs:

```txt
creator
asset_mints[]
weights_bps[]
initial_seed_usdc
fee_config
```

Checks:

```txt
asset_count >= 2
asset_count <= 5
no duplicate assets
sum(weights_bps) == 10000
each weight >= global_min_weight_bps
each asset creation_enabled == true
each weight <= asset.max_weight_bps
```

## Mint

Mints DTF tokens with USDC.

Inputs:

```txt
market_id
usdc_amount_in
route_plan[]
min_outs[]
```

Checks:

```txt
market active
each asset mint_enabled
allocation >= 1 USDC
allocation <= max_trade_usdc
route approved
price impact within policy
actual_received >= min_out
pricing source valid
```

## Redeem

Redeems DTF tokens back to USDC.

Inputs:

```txt
market_id
dtf_amount_in
route_plan[]
min_usdc_out
```

Checks:

```txt
market active or redeem allowed
each asset redeem_enabled
route approved
actual_usdc_received >= min_usdc_out
pricing deviation within policy
```

## SetAssetExecutionPolicy

Updates asset-level execution policy.

## SetAssetExecutionFlags

Updates asset execution flags.

Recommended emergency setting:

```txt
creation_enabled = false
mint_enabled = false
redeem_enabled = true
rebalance_enabled = false
```

## RegisterApprovedRoute

Registers an approved CPI route.

## SetPricingSource

Registers or updates pricing source.

## Rebalance

Status: not finalized.

## PauseMarket / UnpauseMarket

Status: likely needed.

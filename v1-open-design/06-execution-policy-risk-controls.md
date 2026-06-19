# Execution Policy / Risk Controls

## 1. Goal

Axis v1 Open Version should avoid market-level TVL caps.

Risk should be controlled through:

- composition rules
- asset-level execution policies
- per-transaction trade limits
- approved routes
- pricing checks
- actual balance delta checks
- asset-level enable / disable flags

## 2. Open Version Decision

```txt
market_tvl_cap = none
```

No DTF-level market TVL cap is used in the Open Version.

Instead, the protocol limits what can happen in a single transaction.

## 3. Composition Rules

```txt
min_assets_per_dtf = 2
max_assets_per_dtf = 5
duplicate_asset = false
total_weight_bps = 10000
global_min_weight_bps = 100
hard_min_allocation_usdc = 1
market_tvl_cap = none
```

One-asset DTFs are not allowed.

## 4. Minimum Allocation

Every asset allocation during mint must be at least 1 USDC.

```txt
asset_allocation_usdc_i = mint_amount_usdc × weight_bps_i / 10000

require asset_allocation_usdc_i >= 1 USDC
```

Examples:

```txt
1% asset included  -> minimum mint = 100 USDC
2% asset included  -> minimum mint = 50 USDC
5% asset included  -> minimum mint = 20 USDC
10% asset included -> minimum mint = 10 USDC
```

## 5. Asset Execution Policy

Each asset has an execution policy.

```txt
AssetExecutionPolicy {
  asset_mint

  creation_enabled
  mint_enabled
  redeem_enabled
  rebalance_enabled

  hard_min_allocation_usdc

  max_trade_usdc
  max_weight_bps
  max_price_impact_bps

  pricing_requirement
  max_pricing_deviation_bps

  approved_route_required
  manual_review_required
}
```

## 6. Default Policy Presets

These are default presets, not immutable hardcoded categories. Individual assets may override them.

```txt
Core / Stable / LST:
max_trade_usdc = 50,000
max_weight_bps = 10000
max_price_impact_bps = 100

Major / Blue-chip:
max_trade_usdc = 10,000
max_weight_bps = 5000
max_price_impact_bps = 200

Volatile / Mid-cap:
max_trade_usdc = 1,000
max_weight_bps = 2500
max_price_impact_bps = 300

Long-tail:
max_trade_usdc = 250
max_weight_bps = 1000
max_price_impact_bps = 500

StockToken / Restricted:
max_trade_usdc = 1,000
max_weight_bps = 2000
max_price_impact_bps = 300
```

## 7. Runtime Mint Checks

For each asset in the DTF:

```txt
asset_allocation_usdc_i = mint_amount_usdc × weight_bps_i / 10000
```

Require:

```txt
asset_allocation_usdc_i >= 1 USDC
asset_allocation_usdc_i <= asset.max_trade_usdc
weight_bps_i <= asset.max_weight_bps
expected_price_impact_bps <= asset.max_price_impact_bps
actual_received_i >= min_out_i
pricing_deviation_bps <= max_pricing_deviation_bps
```

## 8. Single Mint Max

The maximum single mint size is naturally constrained by the weakest asset in the basket.

```txt
single_mint_max_usdc = min(asset.max_trade_usdc / asset.weight_fraction)
```

Example:

```txt
DTF:
90% SOL
10% Long-tail meme

Long-tail max_trade_usdc = 250

single_mint_max_usdc = 250 / 0.10 = 2,500 USDC
```

## 9. Redeem Policy

Redeem should remain available whenever possible.

If an asset becomes risky, Axis should be able to block new creation and minting while preserving exits.

```txt
creation_enabled = false
mint_enabled = false
redeem_enabled = true
rebalance_enabled = false
```

## 10. Asset Execution Flags

Each asset has independent flags:

```txt
AssetExecutionFlags {
  creation_enabled
  mint_enabled
  redeem_enabled
  rebalance_enabled
}
```

## 11. Policy Presets vs Risk Tiers

Risk tiers may be introduced later as policy presets.

However, the initial design should not make risk tier the core protocol primitive. The core primitive should be the asset execution policy.

```txt
Core primitive:
  AssetExecutionPolicy

Optional later abstraction:
  RiskTier -> default policy preset
```

# Account Spec

This document defines the initial conceptual account model for Axis v1. Exact byte layout is implementation-specific and should be finalized during program design.

## ProtocolConfig

```txt
ProtocolConfig {
  authority
  pause_authority
  default_usdc_mint
  protocol_treasury
  asset_registry_authority
  route_registry_authority
  pricing_registry_authority
}
```

## DTFMarket

```txt
DTFMarket {
  market_id
  creator
  dtf_mint
  reserve_authority
  asset_count
  status
  total_weight_bps
  fee_config
  created_at
}
```

## MarketAssetWeight

```txt
MarketAssetWeight {
  market_id
  asset_mint
  weight_bps
  reserve_token_account
}
```

## AssetRegistry

```txt
AssetRegistry {
  asset_mint
  symbol
  name
  category
  decimals
  is_universe_asset
  is_placeholder
}
```

## AssetExecutionPolicy

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

## ApprovedRoute

```txt
ApprovedRoute {
  route_id
  asset_mint
  quote_mint
  direction
  venue_type
  venue_program_id
  pool_id
  input_mint
  output_mint
  max_trade_usdc
  max_price_impact_bps
  max_accounts
  requires_token_2022
  enabled
}
```

## PricingSource

```txt
PricingSource {
  pricing_source_id
  asset_mint
  source_type
  source_account
  quote_mint
  max_staleness_slots
  max_deviation_bps
  enabled
}
```

## FeeConfig

```txt
FeeConfig {
  mint_fee_bps
  redeem_fee_bps
  creator_share_bps
  protocol_share_bps
  fee_destination
}
```

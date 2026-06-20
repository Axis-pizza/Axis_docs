# Math Spec

## 1. Constants

```txt
TOTAL_WEIGHT_BPS = 10000
GLOBAL_MIN_WEIGHT_BPS = 100
HARD_MIN_ALLOCATION_USDC = 1
INITIAL_NAV_USDC = 1
```

## 1b. Mint Fee Deduction

Mint fee is charged on gross user USDC input before reserve composition.

```txt
mint_fee_usdc = gross_user_usdc_in × MINT_FEE_BPS / 10000
net_usdc_for_composition = gross_user_usdc_in - mint_fee_usdc

creator_fee_usdc = mint_fee_usdc × CREATOR_SHARE_BPS / 10000
protocol_fee_usdc = mint_fee_usdc - creator_fee_usdc
```

Only `net_usdc_for_composition` is used for reserve composition. Mint fee is excluded from reserve value and NAV.

## 2. Asset Allocation

Allocation is computed from net USDC after mint fee deduction, not gross user input.

```txt
asset_allocation_usdc_i = net_usdc_for_composition × weight_bps_i / 10000
```

Requirement:

```txt
asset_allocation_usdc_i >= 1 USDC
```

## 3. Minimum Mint Size

The 1 USDC allocation rule applies to `net_usdc_for_composition`, so the minimum is expressed against net composition value:

```txt
minimum_net_composition_usdc = max_i(1 USDC / weight_fraction_i)
```

Where:

```txt
weight_fraction_i = weight_bps_i / 10000
```

Examples (net composition value required):

```txt
1% asset  -> minimum net composition = 100 USDC
2% asset  -> minimum net composition = 50 USDC
5% asset  -> minimum net composition = 20 USDC
10% asset -> minimum net composition = 10 USDC
```

TODO: gross minimum mint input is slightly higher than the net composition value because of the mint fee (`net = gross × (1 - MINT_FEE_BPS/10000)`). Exact gross minimum figures are not pinned here to avoid restating fee-derived numbers; derive from the confirmed `MINT_FEE_BPS` if a gross threshold is needed.

## 4. Single Mint Max Size

```txt
single_mint_max_usdc = min_i(asset.max_trade_usdc_i / weight_fraction_i)
```

Example:

```txt
Long-tail weight = 10%
Long-tail max_trade_usdc = 250

single_mint_max_usdc = 250 / 0.10 = 2,500 USDC
```

## 5. Asset Value

```txt
asset_value_usdc_i = reserve_balance_i × approved_price_i
```

## 6. Reserve Value

```txt
reserve_value_usdc = Σ asset_value_usdc_i
```

## 7. NAV

```txt
nav_per_dtf = reserve_value_usdc / total_dtf_supply
```

If supply is zero:

```txt
nav_per_dtf = 1 USDC
```

## 8. Minted DTF

```txt
actual_added_value_usdc = Σ(actual_received_asset_i × approved_price_i)
```

```txt
minted_dtf = actual_added_value_usdc / pre_trade_nav
```

## 9. Redeem Share

```txt
redeem_share = dtf_amount_in / total_supply_before
```

## 10. Redeem Asset Amount

```txt
redeem_asset_amount_i = reserve_balance_i × redeem_share
```

## 11. Actual USDC Out

```txt
actual_usdc_received = post_usdc_balance - pre_usdc_balance
```

```txt
user_usdc_out = actual_usdc_received - redeem_fee
```

For v1, `REDEEM_FEE_BPS = 0`, so there is no explicit Axis exit fee:

```txt
redeem_fee = 0
user_usdc_out = actual_usdc_received
```

Execution spread / slippage / price impact are separate from Axis fees and are bounded by `min_usdc_out`.

## 12. Pricing Deviation

```txt
deviation_bps = abs(execution_price - reference_price) / reference_price × 10000
```

Requirement:

```txt
deviation_bps <= max_pricing_deviation_bps
```

## 13. Price Impact

```txt
price_impact_bps <= max_price_impact_bps
```

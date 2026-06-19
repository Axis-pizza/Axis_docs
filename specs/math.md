# Math Spec

## 1. Asset Allocation

```txt
asset_allocation_usdc_i = mint_amount_usdc × weight_bps_i / 10000
```

```txt
asset_allocation_usdc_i >= hard_min_allocation_usdc
```

For v1:

```txt
hard_min_allocation_usdc = 1 USDC
```

## 2. Reserve Value

```txt
asset_value_usdc_i = reserve_balance_i × approved_price_i
```

```txt
reserve_value_usdc = Σ asset_value_usdc_i
```

## 3. NAV

```txt
nav_per_dtf = reserve_value_usdc / total_dtf_supply
```

If supply is zero:

```txt
initial_nav = 1 USDC
```

## 4. Minted DTF

```txt
actual_added_value_usdc = Σ(actual_received_asset_i × approved_price_i)
```

```txt
minted_dtf = actual_added_value_usdc / pre_trade_nav
```

## 5. Redeem Share

```txt
redeem_share = dtf_amount_in / total_supply_before
```

## 6. Redeem Asset Amount

```txt
redeem_asset_amount_i = reserve_balance_i × redeem_share
```

## 7. USDC Out

```txt
actual_usdc_received = post_usdc_balance - pre_usdc_balance
```

```txt
user_usdc_out = actual_usdc_received - redeem_fee
```

## 8. Single Mint Max

```txt
single_mint_max_usdc = min(asset.max_trade_usdc / asset.weight_fraction)
```

Where:

```txt
asset.weight_fraction = weight_bps_i / 10000
```

## 9. Price Deviation

```txt
deviation_bps = abs(execution_price - reference_price) / reference_price × 10000
```

```txt
deviation_bps <= max_pricing_deviation_bps
```

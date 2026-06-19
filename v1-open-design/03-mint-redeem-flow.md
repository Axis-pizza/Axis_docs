# Mint / Redeem Flow

## 1. Design Goal

Axis v1 must support USDC-based mint and redeem.

- Mint: user deposits USDC and receives DTF tokens.
- Redeem: user burns DTF tokens and receives USDC.

The underlying composition and unwind happen inside the Axis program through controlled CPI execution.

## 2. Mint Flow

```txt
1. User deposits USDC
2. Axis validates the DTF market
3. Axis validates market asset weights
4. Axis loads AssetExecutionPolicy for each asset
5. Axis computes target allocation per asset
6. Axis verifies allocation constraints
7. Axis executes CPI swaps through approved venue adapters
8. Underlying assets are deposited into reserve token accounts
9. Axis reads actual balance deltas
10. Axis values actual received assets using approved pricing sources
11. Axis computes actual_added_value_usdc
12. Axis computes minted_dtf using pre-trade NAV
13. Axis mints DTF tokens to the user
```

## 3. Mint Accounting

```txt
asset_allocation_usdc_i = mint_amount_usdc × weight_bps_i / 10000
```

```txt
require asset_allocation_usdc_i >= 1 USDC
```

```txt
actual_added_value_usdc = Σ(actual_received_asset_i × approved_price_i)
```

```txt
minted_dtf = actual_added_value_usdc / pre_trade_nav
```

If `total_supply == 0`:

```txt
initial_nav = 1 USDC
```

## 4. Redeem Flow

```txt
1. User submits DTF tokens for redemption
2. Axis validates the DTF market
3. Axis computes redeem share
4. Axis computes pro-rata reserve asset amounts
5. Axis validates route and execution policy
6. Axis executes CPI swaps from reserve assets back to USDC
7. Axis reads actual USDC balance delta
8. Axis verifies min_usdc_out and pricing deviation
9. Axis deducts redeem fee if applicable
10. Axis transfers USDC to user
11. Axis burns or accounts for redeemed DTF tokens
```

## 5. Redeem Accounting

```txt
redeem_share = dtf_amount_in / total_supply_before
```

```txt
redeem_asset_amount_i = reserve_balance_i × redeem_share
```

```txt
actual_usdc_received = post_usdc_balance - pre_usdc_balance
```

```txt
user_usdc_out = actual_usdc_received - redeem_fee
```

## 6. All-or-Nothing Execution

Mint and redeem should be all-or-nothing.

```txt
Any swap fails -> entire mint/redeem fails
Any min_out check fails -> entire mint/redeem fails
Any policy violation -> entire mint/redeem fails
```

## 7. Actual Balance Delta as Source of Truth

Axis does not trust off-chain quotes as final accounting truth.

```txt
actual_balance_delta = post_balance - pre_balance
```

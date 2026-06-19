# Pricing / NAV Accounting

## 1. Goal

Axis DTFs are backed by actual reserve assets.

Therefore, the protocol needs a robust way to value underlying reserves during:

- market creation
- mint
- redeem
- UI display
- risk checks
- future rebalance logic

## 2. Pricing Source Registry

Axis v1 should use a **Pricing Source Registry**, not only an Oracle Registry.

Pricing sources may include:

```txt
ExternalOracle
DexTwap
DexSpot
StablePeg
LstExchangeRate
StockTokenOracle
```

## 3. Pricing Source Types

### ExternalOracle

Used for:

- SOL
- major assets
- stable assets
- LSTs
- StockTokens
- high-confidence assets

### DexTwap

Used for:

- meme blue-chip
- meme mid-cap
- assets without external oracle but with sufficiently liquid approved pools

### DexSpot

Used only with strict caps.

Used for:

- long-tail memes
- experimental assets
- Pump-style assets

### StablePeg

Used for:

- USDC
- USDT
- USDG
- USD1

Must include depeg checks.

### LstExchangeRate

Used for LST assets.

```txt
LST value = LST/SOL exchange rate × SOL/USD price
```

### StockTokenOracle

Used for:

- tokenized equities
- xStocks
- ETF-like assets

StockToken pricing should require manual review and stronger oracle / issuer assumptions.

## 4. NAV Formula

```txt
asset_value_usdc_i = reserve_balance_i × approved_price_i
```

```txt
reserve_value_usdc = Σ asset_value_usdc_i
```

```txt
nav_per_dtf = reserve_value_usdc / total_dtf_supply
```

If supply is zero during initial seeding:

```txt
initial_nav = 1 USDC
```

## 5. Mint Accounting

```txt
pre_trade_nav = reserve_value_before / total_supply_before
```

```txt
actual_added_value_usdc = Σ(actual_received_asset_i × approved_price_i)
```

```txt
minted_dtf = actual_added_value_usdc / pre_trade_nav
```

## 6. Redeem Accounting

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

## 7. Pricing Tiers

### Oracle Required

Used for:

- stable
- LST
- StockToken
- major assets

### TWAP Allowed

Used for:

- meme blue-chip
- meme mid-cap
- selected DeFi assets

### Spot with Strict Caps

Used for:

- long-tail memes
- experimental assets

### Disabled Until Pricing Source

Used when no reliable pricing source exists.

## 8. UI Price vs Accounting Price

UI prices and accounting prices must be separated.

```txt
UI display price:
  - fast
  - user-friendly
  - approximate

On-chain accounting price:
  - approved
  - verified
  - deterministic
  - used for mint/redeem
```

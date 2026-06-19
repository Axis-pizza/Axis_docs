# DTF Model

## 1. Definition

A **DTF**, or **Dex Traded Fund**, is a tradable position token backed by a basket of actual underlying reserve assets.

Canonical Japanese definition:

```txt
DTFは、作成者が定義した目標ポジションに対応する裏側資産を持ち、その資産価値と市場設計に基づいて売買されるポジショントークンである。
```

Canonical English definition:

```txt
A DTF is a tradable position token backed by underlying reserve assets corresponding to a creator-defined target position, and traded based on those reserve assets and the market design around them.
```

## 2. What a DTF Is

A DTF represents a view, thesis, or position.

Examples:

- Solana AI DTF
- Meme Blue-chip DTF
- Solana DeFi DTF
- Crypto Equities DTF
- DePIN DTF
- LST / Yield DTF
- Political Meme DTF
- Risk-on Meme Rotation DTF

## 3. What a DTF Is Not

A DTF is not:

- a synthetic price token
- a pure USDC wrapper
- a frontend-only multi-swap basket
- a one-asset wrapped token
- a passive Google Sheet of token weights

Axis v1 requires actual reserve backing.

## 4. Asset Count

```txt
min_assets_per_dtf = 2
max_assets_per_dtf = 5
```

One-asset DTFs are not allowed. A one-asset DTF would behave more like a wrapped token or swap proxy than a tradable view or position token.

## 5. Target Weights vs Actual Reserves

Each DTF has creator-defined target weights.

However, accounting is based on actual reserve balances, not target weights.

```txt
Target weights define intended composition.
Actual reserves define accounting value.
```

## 6. Composition Rules

```txt
min_assets_per_dtf = 2
max_assets_per_dtf = 5
duplicate_asset = false
total_weight_bps = 10000
global_min_weight_bps = 100
hard_min_allocation_usdc = 1
market_tvl_cap = none
```

## 7. Minimum Allocation Rule

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

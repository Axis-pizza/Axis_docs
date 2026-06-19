# Definitions & Decision Log

## 1. DTF

### Definition

A **DTF**, or **Dex Traded Fund**, is a tradable position token backed by a basket of actual underlying reserve assets.

Japanese canonical definition:

```txt
DTFは、作成者が定義した目標ポジションに対応する裏側資産を持ち、その資産価値と市場設計に基づいて売買されるポジショントークンである。
```

English canonical definition:

```txt
A DTF is a tradable position token backed by underlying reserve assets corresponding to a creator-defined target position, and traded based on those reserve assets and the market design around them.
```

## 2. Axis Core

Axis Core is the on-chain protocol layer responsible for:

```txt
- DTF issuance
- mint
- redeem
- reserve custody
- accounting
- execution policy validation
- pricing validation
- controlled CPI swap execution
```

## 3. Titan Boundary

Titan is not Axis Core.

```txt
Axis Core = issuance, mint, redeem, reserves, accounting
Titan     = routing, distribution, external route layer
```

Decision:

```txt
Axis First. Titan Compatible. Controlled Execution.
```

Axis must work independently even if Titan is not integrated.

## 4. Open Version

Axis v1 Open Version means:

```txt
- no market-level TVL cap
- DTF creation is open within approved asset universe and policy constraints
- risk is controlled at asset and transaction level
```

Decision:

```txt
market_tvl_cap = none
```

## 5. Underlying Reserve Backing

Axis v1 DTFs are backed by actual underlying reserve assets.

Decision:

```txt
DTF reserves hold the actual assets corresponding to the DTF composition.
```

Rejected model:

```txt
USDC-only reserve wrapper with synthetic exposure
```

## 6. Asset Count

Decision:

```txt
min_assets_per_dtf = 2
max_assets_per_dtf = 5
```

Reason:

```txt
1 asset DTF behaves like a wrapped token or swap proxy, not a position basket.
```

Requirement:

```txt
1 asset DTF must be rejected.
```

## 7. Target Weights vs Actual Reserves

Decision:

```txt
Target weights define intended composition.
Actual reserve balances define accounting value.
```

NAV must not be calculated from target weights alone.

## 8. Mint Input and Redeem Output

Decision:

```txt
Mint input  = USDC
Redeem output = USDC
```

Underlying composition and unwind happen through Axis controlled CPI execution.

## 9. Initial NAV

Decision:

```txt
initial_nav = 1 USDC
```

Used when:

```txt
total_supply == 0
```

## 10. Minimum Allocation

Decision:

```txt
hard_min_allocation_usdc = 1 USDC
```

Each asset allocation during mint must be at least 1 USDC.

```txt
asset_allocation_usdc_i = mint_amount_usdc × weight_bps_i / 10000
require asset_allocation_usdc_i >= 1 USDC
```

## 11. Global Minimum Weight

Decision:

```txt
global_min_weight_bps = 100
```

This means the minimum weight for any DTF component is 1%.

## 12. Risk Control Model

Decision:

```txt
Do not control risk with market TVL caps.
Control risk with per-asset and per-transaction execution policy.
```

Risk controls:

```txt
- asset max weight
- asset max trade size per transaction
- max price impact
- min_out
- approved route
- pricing deviation
- actual balance delta check
- asset enable / disable flags
```

## 13. Swap Execution Model

Decision:

```txt
Route discovery and account assembly may happen off-chain.
Execution and verification happen on-chain.
```

Axis should not become a general-purpose DEX aggregator.

## 14. First CPI Spike

Decision:

```txt
Orca Whirlpool is the first CPI spike candidate.
```

Reason:

```txt
- mature venue
- good for majors / stable / LST
- direct technical access to Orca team
- useful for early feasibility validation
```

Architecture remains adapter-based.

## 15. Jupiter Role

Decision:

```txt
Jupiter is useful for route discovery and market research.
Jupiter route availability is not equivalent to Axis CPI readiness.
```

Distinction:

```txt
Jupiter route_available = likely swappable through Jupiter
Axis approved CPI route = executable by Axis program through approved adapter
```

## 16. Pricing Source Registry

Decision:

```txt
Use Pricing Source Registry, not only Oracle Registry.
```

Supported source types:

```txt
ExternalOracle
DexTwap
DexSpot
StablePeg
LstExchangeRate
StockTokenOracle
```

## 17. Asset Universe

Decision:

```txt
Axis v1 targets a curated 500-asset universe.
```

This is not simply top 500 by liquidity.

Asset universe categories:

```txt
Meme Blue-chip          50
Meme Mid-cap            100
Meme Long-tail          150
StockToken              75
Solana DeFi / Infra     50
AI / DePIN / Agent      40
Stable / LST / Yield    25
Experimental            10
Total                   500
```

## 18. Asset Enable / Disable Flags

Decision:

```txt
AssetExecutionFlags {
  creation_enabled
  mint_enabled
  redeem_enabled
  rebalance_enabled
}
```

Emergency recommended setting:

```txt
creation_enabled = false
mint_enabled = false
redeem_enabled = true
rebalance_enabled = false
```

Goal:

```txt
Block new risk while preserving exit paths.
```

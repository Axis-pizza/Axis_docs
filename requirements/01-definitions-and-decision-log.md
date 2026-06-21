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

Allocation is computed from net USDC after mint fee deduction, not gross user input.

```txt
asset_allocation_usdc_i = net_usdc_for_composition × weight_bps_i / 10000
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

## 19. Fee Model

Creator fee and protocol fee are required Axis v1 protocol concepts from launch, not future add-ons. See `13-fee-model-requirements.md`.

Decision:

```txt
mint_fee_bps = 100
redeem_fee_bps = 0

creator_share_bps = 4000
protocol_share_bps = 6000

max_mint_fee_bps = 300
max_redeem_fee_bps = 0
```

Fee model rules:

```txt
- fees are charged on the USDC side
- mint fee is deducted before reserve composition (net_usdc_for_composition is used)
- redeem has no explicit Axis exit fee
- fees are claim-based, not immediately transferred
- creators cannot customize fee bps per market
- market fee config is immutable after market creation
- fee custody is separate from reserve custody
- accrued fees are excluded from NAV and not counted as reserve backing
```

## 20. Validation Path

Decision:

```txt
Public Devnet is not a mandatory validation path or launch gate.
```

Axis v1 readiness is based on validation evidence, not on public Devnet runtime. Pre-mainnet validation relies on local tests, LiteSVM tests, local validator tests, mainnet-fork / cloned-account tests, production venue CPI integration tests, app contract integration smoke tests, and guarded mainnet candidate controls. Public Devnet may be used opportunistically. See `12-pre-mainnet-validation-requirements.md`.

## 21. Launch-Day Secondary Market Surface

Decision:

```txt
Axis v1 launches with a secondary-market surface from day one.
```

Created DTFs must be usable as tradable and distributable tokens, including in partner or sponsor campaigns. The launch requirement is a discoverable Axis-operated surface that presents the market and available secondary-liquidity references; it is not a requirement that Axis-native auction/JIT liquidity be production-enabled for every market on day one.

See `18-secondary-market-and-clear-correction-requirements.md`.

## 22. Public Third-Party DTF/USDC Pools

Decision:

```txt
Third parties may create public DTF/USDC pools on external venues, including Orca and Raydium.
```

These pools are external liquidity. Axis may index or display verified pool references, but they are not part of Axis reserve custody or Axis Core accounting.

External-pool policy:

```txt
- public external pools are not Axis-native liquidity
- public external pools are not protected by Axis LVR mitigation
- Axis must not market or label public external pools as auction-enabled unless the separate Axis Auction Program controls the relevant settlement path
- external pool prices are not reserve NAV and do not replace the Pricing Source Registry
```

## 23. Scope of Axis-Native LVR Mitigation

Decision:

```txt
Axis-native LVR mitigation applies only to liquidity using Axis-controlled auction/JIT settlement.
```

The LVR-mitigated surface is therefore not a generic property of the DTF token or of any DTF/USDC pool. It is a property of an explicitly activated Axis-native liquidity configuration.

## 24. Preferred Axis-Native Secondary Liquidity Path

Decision:

```txt
Preferred research path = Axis-controlled JIT liquidity on Orca.
First settlement venue candidate = Orca Whirlpool.
```

This choice applies to the Axis-controlled JIT liquidity / NAV correction design, not to the existing mint/redeem venue order. Raydium remains an important underlying mint/redeem execution fallback and may host external liquidity, but it is not the first settlement target for the Axis-controlled JIT research path.

## 25. Architectural Compatibility and Activation

Decision:

```txt
All DTF markets are architecturally compatible with Axis-controlled JIT liquidity and NAV Correction Auctions.
```

Architectural compatibility is not activation. A market may be enabled only after its pool availability, pricing freshness, route support, auction-program support, account/compute feasibility, and safety validation have passed the applicable readiness gates. Until then, the market may use the launch-day secondary-market surface and external liquidity without an Axis LVR-mitigation claim.

## 26. Auction Program Boundary

Decision:

```txt
ClearCorrection and NAV Correction Auction logic belong in a separate Axis Auction Program.
```

Axis Core remains responsible for:

```txt
- DTF market creation
- mint and redeem
- reserve custody and reserve accounting
- NAV calculation and pricing-source validation
- primary-flow fee accounting
```

The Axis Auction Program is responsible for auction state, winner authorization, correction bounds, correction settlement coordination, auction payment settlement, and auction events. It must not treat auction revenue as DTF reserve backing.

## 27. Correction Settlement Atomicity

Decision:

```txt
Preferred ClearCorrection execution mode = one transaction.
```

If a one-transaction design is not feasible, Jito bundles may be considered only as a fallback after technical validation demonstrates ordered atomic execution and prevents non-winner interception. A multi-transaction fallback without those properties is not acceptable for a correction settlement path.

## 28. Auction Revenue Separation

Decision:

```txt
Auction revenue is accounted for separately from DTF reserves and mint/redeem fees.
```

The expected accounting boundary is an `AuctionRevenueVault` or an equivalent dedicated account model. Exact account layout, revenue recipients, fee splits, bid format, and claim mechanics remain configurable/TBD. Auction revenue is excluded from DTF reserve value and NAV.

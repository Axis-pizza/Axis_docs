## 8. Asset Universe and Supported Asset Enablement Requirements

### 8.1 Purpose

Axis v1 must define a clear asset enablement process for determining which assets can be used as reserve assets in DTF markets.

The goal of the asset universe process is not to create a perfectly balanced thematic asset list, nor to automatically classify every Solana asset. The goal is to identify which assets are actually usable for Axis v1 based on liquidity, route availability, pricing feasibility, token safety, and operational risk.

Asset discovery should be treated as an input to human review, not as an automatic launch approval mechanism.

### 8.2 Liquidity-First Asset Discovery

Axis v1 asset discovery must be liquidity-first.

The initial asset universe must not be selected by fixed category quotas. Category labels such as meme, DeFi, AI/DePIN, stablecoin, wrapped asset, or stock token are useful for review and product presentation, but they must not be the primary selection mechanism.

The primary selection criteria should be:

1. Whether the asset has a direct USDC pool or a practical USDC route
2. Total available liquidity / TVL
3. 24h trading volume
4. Number of supported venues where the asset is available
5. Whether the asset is available through Axis-supported execution venues
6. Whether the token mint, decimals, token program, and extensions are safe to support
7. Whether a reliable pricing source or pricing policy can be defined
8. Whether the asset is strategically relevant for DTF creation

Category is a secondary review label, not the first filter.

### 8.3 Discovery Output Is Not Launch Approval

All assets discovered by scripts must initially be treated as `CANDIDATE_UNIVERSE`.

No discovery script may automatically mark an asset as launch-ready, mint-enabled, redeem-enabled, or creation-enabled.

The default flags for auto-discovered assets must be:

```txt
launchReadiness = CANDIDATE_UNIVERSE
creationEnabled = false
mintEnabled = false
redeemEnabled = false
pricingTier = DISABLED_UNTIL_PRICING_SOURCE
routeStatus = UNKNOWN
```

Assets can only move from candidate status to supported status after route, pricing, mint, and risk review.

### 8.4 Asset Support Statuses

Axis v1 should use the following asset support statuses:

```txt
SUPPORTED_CANDIDATE_V1
INCLUDE_AFTER_ROUTE
SPIKE_REQUIRED
MANUAL_REVIEW_REQUIRED
ROUTE_REQUIRED
PRICING_REQUIRED
MINT_REVIEW_REQUIRED
DEFERRED
REJECTED_V1
```

#### SUPPORTED_CANDIDATE_V1

Assets that appear suitable for Axis v1 support and should proceed to route, pricing, and implementation validation.

This status does not mean the asset is launch-ready. It means the asset is a priority candidate for implementation work.

#### INCLUDE_AFTER_ROUTE

Assets that are strategically important and likely useful for Axis v1, but still require direct USDC route validation or ApprovedRoute validation before they can be enabled.

#### SPIKE_REQUIRED

Assets that are important or liquid, but require additional technical or risk investigation before they can be considered for support.

This includes, but is not limited to:

* xStocks / stock tokens
* Token-2022 assets
* assets with scaled UI amount behavior
* assets with transfer hooks or permanent delegates
* wrapped assets
* bridge assets
* newer stablecoins
* AI/DePIN assets with unclear issuer or pricing risk
* assets with high liquidity but unclear category or risk profile

#### MANUAL_REVIEW_REQUIRED

Assets that have meaningful liquidity but cannot be safely classified or approved automatically.

These assets should be reviewed manually for category, issuer, pricing, route, token safety, and legal/risk considerations.

#### DEFERRED / REJECTED_V1

Assets that are not suitable for Axis v1 launch scope due to low relevance, insufficient safety, unclear mint, poor pricing, excessive risk, or lack of route support.

### 8.5 Supported Asset List v0

Based on the current liquidity-first review, Axis v1 should proceed with the following supported asset candidate list v0.

This is not a final launch-ready list. This is the initial implementation and validation list.

#### Include / direct or mostly clear

```txt
SOL
USDT
JitoSOL
JupUSD
RAY
JLP
KMNO
JUP
mSOL
Fartcoin
POPCAT
BONK
```

#### Include after route validation

```txt
JTO
ORCA
PYTH
DRIFT
JupSOL
$WIF
BOME
MEW
PONKE
FWOG
GOAT
```

Together, these form the initial Axis v1 supported asset candidate set:

```txt
Total supported v0 candidates: 23
```

### 8.6 Assets Requiring Spike Before Support

The following asset groups must not be treated as supported by default, even if they have meaningful liquidity.

They should be routed into `SPIKE_REQUIRED` until their execution, pricing, and safety requirements are validated.

#### Wrapped / bridge assets

Examples:

```txt
cbBTC
whETH
WBTC
TRX
ZEC
XRP
```

These assets may have meaningful liquidity, but they require additional bridge/wrapped asset risk review and pricing policy review.

#### Stablecoin / yield-bearing stable assets

Examples:

```txt
USDS
USDG
PYUSD
USX
USD1
syrupUSDC
```

Stable assets must be reviewed individually. Liquidity alone is not sufficient. Issuer risk, redemption assumptions, depeg risk, transfer behavior, and pricing policy must be reviewed.

#### AI / DePIN assets

Examples:

```txt
AAIF
GEOD
RENDER
NOS
HNT
```

These assets may be strategically relevant, but should require pricing, issuer, liquidity, and route validation before support.

#### StockToken / xStocks

Examples:

```txt
SPYx
TSLAx
NVDAx
CRCLx
SPCX
SPCXx
STRCx
QQQx
MSTRx
COINx
GLDx
```

All xStocks and stock tokens must remain `SPIKE_REQUIRED` until the separate StockToken/xStocks requirements are satisfied.

xStocks require special handling for Token-2022, scaled UI amount behavior, token extensions, market-hours behavior, pricing, and Raydium/Orca execution validation.

### 8.7 Unclassified High-Liquidity Assets

Unclassified assets must not be ignored, but they must not be automatically included.

The asset discovery process should maintain a top-N unclassified review queue. For v1, the first review batch should focus on the top 30 unclassified assets by liquidity and volume.

Examples of assets that require manual review include:

```txt
CARDS
ONyc
CASH
HYPE
BORG
PUMP
2Z
GRASS
ME
VCHF
VNXAU
NEAR
wYLDS
PENGU
GIGA
```

These assets may become supported, spike, deferred, or rejected after manual review.

### 8.8 Launch-Ready Promotion Gates

An asset may only move from candidate or spike status to launch-ready status after satisfying all of the following gates:

1. Canonical mint is confirmed
2. Decimals and token program are confirmed
3. Token extensions are reviewed
4. Direct USDC route or ApprovedRoute is validated
5. Venue accounts are validated
6. Pricing tier is assigned
7. Route simulation passes
8. Mint path is tested
9. Redeem path is tested
10. Asset-specific risk notes are reviewed
11. Asset is manually approved for launch enablement

No asset may be launch-ready based only on liquidity ranking.

### 8.9 Implementation Principle

Axis v1 asset universe construction must be liquidity-first and review-gated.

The correct process is:

```txt
Discover liquid assets
↓
Rank by liquidity and route feasibility
↓
Separate supported candidates, spike candidates, manual review, and deferred assets
↓
Validate routes, pricing, token safety, and mint/redeem execution
↓
Manually approve launch enablement
```

The incorrect process is:

```txt
Choose fixed category quotas
↓
Fill each category
↓
Assume the result is supportable
```

Category balance may be useful for product design and marketing, but it must not determine protocol asset support.

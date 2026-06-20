# Asset Universe Requirements

## 1. Overview

Axis v1 targets a curated 500-asset universe.

The universe is designed for narrative and position construction, not simply liquidity ranking.

## 2. Target Allocation

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

## 3. Requirements

### ASSET-001: Asset Universe must be curated

Assets should be selected for usefulness in DTF creation.

Selection axes:

```txt
- Narrative Value
- Execution Feasibility
- Safety / Legibility
```

### ASSET-002: Asset Universe inclusion must be separate from execution readiness

```txt
Asset Universe inclusion != Launch-ready execution
```

Acceptance criteria:

```txt
- asset can exist in universe but be marked ROUTE_REQUIRED
- asset can exist in universe but be marked PRICING_REQUIRED
- asset can exist in universe but be marked MINT_REQUIRED
```

### ASSET-003: Asset readiness statuses must be tracked

Recommended statuses:

```txt
LAUNCH_READY
SPIKE_CANDIDATE
ROUTE_REQUIRED
PRICING_REQUIRED
MINT_REQUIRED
OPEN_SLOT
DEFERRED
```

TODO: `DEVNET_SPIKE_CANDIDATE` was renamed to `SPIKE_CANDIDATE` because public Devnet is no longer a mandatory validation path (see `12-pre-mainnet-validation-requirements.md`). A spike now means local / fork-based validation. Confirm the final status name.

### ASSET-004: Route readiness must be tracked separately

Recommended route statuses:

```txt
READY
SPIKE_REQUIRED
JUPITER_ONLY
NO_ROUTE
UNKNOWN
```

### ASSET-005: Pricing tier must be tracked for each asset

Pricing tiers:

```txt
ORACLE_REQUIRED
TWAP_ALLOWED
SPOT_WITH_STRICT_CAPS
DISABLED_UNTIL_PRICING_SOURCE
```

### ASSET-006: StockToken assets must require manual review

Acceptance criteria:

```txt
- manual_review_required=true
- spot-only pricing prohibited
- issuer / underlying metadata required
```

### ASSET-007: Experimental assets must have strict execution limits

Acceptance criteria:

```txt
- low max_trade_usdc
- low max_weight_bps
- strict pricing requirements
```

### ASSET-008: Asset metadata should support narrative grouping

Recommended metadata:

```txt
symbol
name
mint
category
sub_category
narrative_tags[]
route_status
pricing_tier
launch_readiness
manual_review_required
```

## 4. Example DTFs Used to Backsolve Asset Universe

```txt
Solana Meme Index
Pump Legends
Political Memes
AI Agents
Solana DeFi
Solana Infra
DePIN
Crypto Equities
Magnificent 7
Degenerate Longtail
Stable / LST Yield
```

## 5. Issue Candidates

```txt
- Define AssetRegistry schema
- Define asset category enum
- Define launch readiness enum
- Define pricing tier enum
- Define route readiness enum
- Build initial Axis 500 CSV/JSON ingestion
- Implement asset registry validation
- Add manual review fields
```

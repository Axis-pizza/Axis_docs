# Asset Universe

## 1. Goal

Axis v1 targets a curated 500-asset universe.

The universe is not simply the top 500 most liquid tokens. It is designed around assets that users can combine into views, theses, narratives, and positions.

## 2. Category Allocation

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

## 3. Selection Philosophy

The Axis 500 Asset Universe should represent assets that users would want to combine into meaningful DTFs.

Example DTFs:

```txt
Solana Meme Index
Political Meme Basket
AI Agent Basket
Solana DeFi Basket
Crypto Equities Basket
Magnificent 7 StockToken Basket
Risk-on Meme Rotation
Blue-chip Solana Basket
DePIN Basket
LST / Yield Basket
RWA / Stable Yield Basket
```

## 4. Selection Criteria

Each asset should be evaluated on three axes:

### Narrative Value

```txt
- recognizable
- belongs to a clear narrative
- useful in an interesting DTF
- likely to be discussed on crypto Twitter
- understandable to users
```

### Execution Feasibility

```txt
- has route availability
- has usable liquidity
- can likely be swapped from USDC or SOL
- price impact is manageable under policy
- compatible with Solana token program assumptions
```

### Safety / Legibility

```txt
- verified or community-recognized
- authority data is acceptable or reviewed
- holder concentration is acceptable
- not obviously fake / duplicate / rug
- issuer and pricing source are clear for StockTokens
```

## 5. Universe vs Execution Readiness

Asset Universe inclusion and execution readiness are separate concepts.

```txt
Axis 500 Universe:
  asset is strategically useful for DTF creation

Execution Ready:
  asset has verified mint, approved route, pricing source, and execution policy
```

Recommended readiness statuses:

```txt
LAUNCH_READY
DEVNET_SPIKE_CANDIDATE
ROUTE_REQUIRED
PRICING_REQUIRED
MINT_REQUIRED
OPEN_SLOT
DEFERRED
```

Recommended route statuses:

```txt
READY
SPIKE_REQUIRED
JUPITER_ONLY
NO_ROUTE
UNKNOWN
```

Important distinction:

```txt
Jupiter route_available = can likely swap through Jupiter
Axis approved CPI route = Axis program can safely execute via approved adapter
```

## 6. Current v0.3 Direction

```txt
1. Classify real mint assets by launch readiness
2. Split route candidates by Orca / Raydium / PumpSwap / Meteora
3. Assign pricing tier to every asset
4. Move placeholders into mint verification queue
5. Fill open slots later
```

This work can be deferred until CPI venue feasibility is clearer.

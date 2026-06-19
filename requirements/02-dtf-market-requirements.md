# DTF Market Requirements

## 1. Overview

A DTF market represents one tradable basket position.

Each market has:

```txt
- creator
- DTF mint
- 2 to 5 underlying assets
- target weights
- reserve token accounts
- fee configuration
- status flags
```

## 2. Market Creation Workflow

```mermaid
flowchart TD
    Creator[Creator] -->|Submit asset mints and weights| Create[CreateMarket Instruction]
    Create --> CountCheck[Validate asset count 2 to 5]
    CountCheck --> DupCheck[Validate no duplicate assets]
    DupCheck --> WeightCheck[Validate total weight = 10000 bps]
    WeightCheck --> MinWeight[Validate each weight >= 100 bps]
    MinWeight --> Registry[Validate each asset in AssetRegistry]
    Registry --> Flags[Validate creation_enabled]
    Flags --> MaxWeight[Validate weight <= asset.max_weight_bps]
    MaxWeight --> Pricing[Validate pricing source exists]
    Pricing --> Routes[Validate route availability requirement]
    Routes --> Market[Create DTFMarket]
    Market --> Mint[Create DTF Mint]
    Market --> Reserves[Create Reserve Token Accounts]
```

## 3. Requirements

### DTF-001: Market must have at least 2 assets

The program must reject any market creation request with fewer than 2 assets.

```txt
asset_count >= 2
```

Acceptance criteria:

```txt
- CreateMarket with 0 assets fails
- CreateMarket with 1 asset fails
- CreateMarket with 2 assets succeeds if all other checks pass
```

### DTF-002: Market must have at most 5 assets

The program must reject any market creation request with more than 5 assets.

```txt
asset_count <= 5
```

Acceptance criteria:

```txt
- 5 asset market can be created
- 6 asset market fails
```

### DTF-003: Duplicate assets are not allowed

The program must reject markets where the same asset mint appears more than once.

Acceptance criteria:

```txt
- [SOL, BONK] passes duplicate check
- [SOL, SOL] fails duplicate check
```

### DTF-004: Total weights must equal 10000 bps

The sum of all target weights must equal 10000 bps.

```txt
sum(weight_bps_i) == 10000
```

Acceptance criteria:

```txt
- 5000 + 5000 passes
- 3333 + 3333 + 3334 passes
- 5000 + 4000 fails
- 6000 + 6000 fails
```

### DTF-005: Each weight must be at least 100 bps

Each component must have weight greater than or equal to 1%.

```txt
weight_bps_i >= 100
```

Acceptance criteria:

```txt
- 100 bps passes
- 99 bps fails
```

### DTF-006: Each weight must be below the asset maximum

Each component must respect the asset-level max weight.

```txt
weight_bps_i <= asset.max_weight_bps
```

Acceptance criteria:

```txt
- Long-tail max_weight_bps = 1000
- 10% long-tail passes
- 15% long-tail fails
```

### DTF-007: Each asset must exist in AssetRegistry

All market assets must be part of the Axis asset universe or otherwise explicitly registered.

Acceptance criteria:

```txt
- registered asset passes
- unknown mint fails
```

### DTF-008: Each asset must be creation-enabled

The program must check:

```txt
asset.creation_enabled == true
```

Acceptance criteria:

```txt
- creation_enabled=false prevents new DTF creation using the asset
- existing DTFs are not automatically deleted
```

### DTF-009: Market-level TVL cap must not be required

Open Version must not require market-level TVL caps.

Acceptance criteria:

```txt
- no market_tvl_cap field is required for normal market operation
- per-transaction constraints still apply
```

### DTF-010: Market must have reserve token accounts

Each underlying asset must have an associated reserve token account controlled by Axis.

Acceptance criteria:

```txt
- reserve account is derived or validated per market and asset
- reserve account mint matches asset mint
- reserve owner/authority is Axis-controlled
```

### DTF-011: Market must have a DTF mint

Each market must have exactly one DTF mint.

Acceptance criteria:

```txt
- DTF mint is unique per market
- DTF mint authority is Axis-controlled
```

### DTF-012: Market should track status

Market status should support at least:

```txt
Created
Active
Paused
Deprecated
```

Acceptance criteria:

```txt
- paused market blocks mint
- paused market may still allow redeem depending on emergency policy
```

## 4. Market State Diagram

```mermaid
stateDiagram-v2
    [*] --> Created
    Created --> Active: initial setup complete
    Active --> Paused: emergency pause
    Paused --> Active: unpause
    Active --> Deprecated: deprecate market
    Paused --> Deprecated: deprecate market
    Deprecated --> [*]
```

## 5. Issue Candidates

```txt
- Implement DTFMarket account
- Implement MarketAssetWeight account
- Implement reserve account derivation
- Implement DTF mint authority model
- Implement CreateMarket validation
- Implement duplicate asset validation
- Implement weight validation
- Implement market status handling
```

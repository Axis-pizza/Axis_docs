# Open Questions

## 1. Fee Model

```txt
- mint_fee_bps
- redeem_fee_bps
- creator_share_bps
- protocol_share_bps
- fee destination
- fee asset
- fee timing
```

## 2. Rebalance Scope

```txt
- v1 or v1.1
- manual or automatic
- creator-only or permissionless
- drift threshold
- rebalance fee
```

## 3. Pricing Implementation

```txt
- exact pricing source account schema
- oracle priority
- DEX TWAP source per venue
- DEX spot source limits
- StockToken pricing source
- stable depeg checks
- LST exchange rate source
```

## 4. Swap CPI Execution

```txt
- Orca Whirlpool CPI feasibility
- compute cost per swap
- maximum realistic swaps per transaction
- Pinocchio / no_std integration path
- Token-2022 / transfer hook handling
- exact adapter interface
```

## 5. Asset Universe

```txt
- finalize 500 assets
- verify placeholders
- fill open slots
- classify launch readiness
- classify approved CPI routes
- classify pricing tiers
```

## 6. Titan Integration

```txt
- integration timing
- quote model
- instruction generation
- exact Titan venue interface
```

## 7. Program Implementation

```txt
- account layout
- instruction layout
- authority model
- upgrade authority
- pause authority
- admin constraints
- error codes
- tests
```

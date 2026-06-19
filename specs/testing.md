# Testing Spec

## 1. Market Creation Tests

```txt
TEST-DTF-001: reject 0 assets
TEST-DTF-002: reject 1 asset
TEST-DTF-003: allow 2 assets
TEST-DTF-004: allow 5 assets
TEST-DTF-005: reject 6 assets
TEST-DTF-006: reject duplicate assets
TEST-DTF-007: reject invalid weight sum
TEST-DTF-008: reject weight below 100 bps
TEST-DTF-009: reject weight above asset max
TEST-DTF-010: reject creation_disabled asset
```

## 2. Mint Tests

```txt
TEST-MINT-001: mint with USDC succeeds
TEST-MINT-002: non-USDC input fails
TEST-MINT-003: allocation calculation correct
TEST-MINT-004: allocation below 1 USDC fails
TEST-MINT-005: allocation above max_trade fails
TEST-MINT-006: mint_disabled asset fails
TEST-MINT-007: unapproved route fails
TEST-MINT-008: min_out failure reverts
TEST-MINT-009: actual balance delta used for accounting
TEST-MINT-010: first mint uses initial NAV = 1
```

## 3. Redeem Tests

```txt
TEST-REDEEM-001: redeem share calculation correct
TEST-REDEEM-002: pro-rata reserve amount correct
TEST-REDEEM-003: wrong DTF mint fails
TEST-REDEEM-004: redeem_disabled asset fails
TEST-REDEEM-005: unapproved route fails
TEST-REDEEM-006: min_usdc_out failure reverts
TEST-REDEEM-007: actual USDC delta used for accounting
TEST-REDEEM-008: DTF burn prevents double redeem
```

## 4. Pricing Tests

```txt
TEST-PRICE-001: missing pricing source fails
TEST-PRICE-002: disabled pricing source fails
TEST-PRICE-003: stale oracle fails
TEST-PRICE-004: pricing deviation above threshold fails
TEST-PRICE-005: NAV uses actual reserve balances
TEST-PRICE-006: target weights are not used as accounting value
```

## 5. CPI Adapter Spike Tests

```txt
TEST-CPI-001: Orca swap CPI can execute in isolation
TEST-CPI-002: Orca swap CPI via Axis adapter can execute
TEST-CPI-003: balance delta measured correctly
TEST-CPI-004: min_out failure reverts
TEST-CPI-005: compute units measured
TEST-CPI-006: account count measured
TEST-CPI-007: multiple swaps in one transaction tested
```

## 6. Emergency Tests

```txt
TEST-ADMIN-001: disable creation blocks CreateMarket
TEST-ADMIN-002: disable mint blocks Mint
TEST-ADMIN-003: redeem remains possible in exit-only mode
TEST-ADMIN-004: pause blocks mint
TEST-ADMIN-005: unauthorized policy update fails
```

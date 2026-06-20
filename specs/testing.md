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
TEST-MINT-003: allocation calculation correct (from net_usdc_for_composition, not gross)
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

## 7. Fee Tests

```txt
TEST-FEE-001: mint fee = gross_user_usdc_in × 100 / 10000
TEST-FEE-002: creator/protocol split = 4000/6000 of mint fee
TEST-FEE-003: net_usdc_for_composition = gross - mint fee, used for composition
TEST-FEE-004: minted_dtf excludes mint fee (based on actual added value)
TEST-FEE-005: redeem charges no explicit fee (user_usdc_out = actual_usdc_received)
TEST-FEE-006: redeem does not accrue creator or protocol fees
TEST-FEE-007: mint_fee_bps > max_mint_fee_bps rejected
TEST-FEE-008: creator_share_bps + protocol_share_bps != 10000 rejected
TEST-FEE-009: creator-customized fee bps rejected
TEST-FEE-010: market fee config immutable after creation
TEST-FEE-011: creator fee claim transfers and decrements accrued balance
TEST-FEE-012: protocol fee claim transfers and decrements accrued balance
TEST-FEE-013: double fee claim rejected
TEST-FEE-014: unauthorized fee claim rejected
TEST-FEE-015: failed mint accrues no fees
TEST-FEE-016: accrued fees excluded from NAV and reserve backing
TEST-FEE-017: fee custody account is separate from reserve accounts
```

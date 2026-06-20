# Fee Model Requirements

## 1. Overview

This document defines the fee model requirements for Axis v1.

Creator fee is a required protocol concept for Axis v1.

Exact fee values are protocol-level configuration decisions, but the protocol must support creator-aligned fee accounting from the beginning.

Axis v1 uses a USDC-side, claim-based creator and protocol fee model.

The purpose of creator fee is to align DTF creators with distribution.

```txt
Creator creates DTF
-> creator shares DTF
-> users mint or trade into DTF exposure
-> creator earns part of the fee flow
-> creator has incentive to distribute the DTF
```

Creator fee must not be treated as a future add-on.

The contract account model, mint flow, redeem flow, app integration, and partner demo story must all assume that creator fee exists from v1 launch.

## 2. Confirmed Fee Model Decisions

Axis v1 uses the following launch fee model:

```txt
mint_fee_bps = 100
redeem_fee_bps = 0

creator_share_bps = 4000
protocol_share_bps = 6000

max_mint_fee_bps = 300
max_redeem_fee_bps = 0
```

Meaning:

```txt
- mint fee is 1%
- redeem has no explicit Axis exit fee
- creator receives 40% of collected mint fees
- protocol receives 60% of collected mint fees
- protocol-level mint fee cap is 3%
- redeem fee cap is 0 for v1
```

Additional confirmed decisions:

```txt
- creator fee is enabled from Axis v1 launch
- protocol fee is enabled from Axis v1 launch
- fees are charged on the USDC side
- fees are accrued and claimed later
- fees are not transferred immediately during mint or redeem
- creators cannot customize fee bps per market
- fee values are fixed by protocol-level configuration or preset
- market fee configuration is immutable after market creation
- fee accounting must not break reserve accounting
- fee accounting must not break NAV accounting
```

## 3. Fee vs Execution Spread

Axis v1 must clearly separate protocol fees from execution spread.

### Explicit Axis Fee

Explicit Axis fee means a creator/protocol fee charged by Axis.

For v1:

```txt
mint_fee_bps = 100
redeem_fee_bps = 0
```

### Execution Spread

Execution spread means the cost or difference caused by market execution.

Execution spread may include:

```txt
- venue spread
- slippage
- price impact
- quote difference
- pool execution loss
- route execution cost
```

Execution spread is not the same as creator/protocol fee.

Redeem has no explicit Axis exit fee, but redeem execution may still produce less USDC than a pre-trade estimate because reserve assets must be unwound through real execution.

Therefore:

```txt
Redeem has no explicit Axis exit fee.
Redeem still requires min_usdc_out protection.
Redeem output is based on actual USDC received after execution.
```

## 4. Protocol-Level Fee Config

Axis v1 must define protocol-level fee parameters.

Example structure:

```rust
pub struct ProtocolFeeConfig {
    pub mint_fee_bps: u16,
    pub redeem_fee_bps: u16,

    pub creator_share_bps: u16,
    pub protocol_share_bps: u16,

    pub max_mint_fee_bps: u16,
    pub max_redeem_fee_bps: u16,

    pub protocol_treasury: Pubkey,
}
```

Launch values:

```txt
mint_fee_bps = 100
redeem_fee_bps = 0

creator_share_bps = 4000
protocol_share_bps = 6000

max_mint_fee_bps = 300
max_redeem_fee_bps = 0
```

Required constraints:

```txt
mint_fee_bps <= max_mint_fee_bps
redeem_fee_bps <= max_redeem_fee_bps
creator_share_bps + protocol_share_bps == 10000
protocol_treasury must be valid
```

Protocol-level fee config may be updated for future markets only if the protocol explicitly supports such an update.

Existing market fee configuration must remain immutable after market creation.

## 5. Market-Level Fee State

Each DTF market must store the fee configuration that applies to that market.

Creators must not be able to customize fee bps per market.

Market-level fee values must be derived from protocol-level fee configuration or a protocol-approved preset at market creation time.

Example structure:

```rust
pub struct MarketFeeState {
    pub creator: Pubkey,
    pub creator_fee_destination: Pubkey,

    pub mint_fee_bps: u16,
    pub redeem_fee_bps: u16,

    pub creator_share_bps: u16,
    pub protocol_share_bps: u16,

    pub accrued_creator_fee_usdc: u64,
    pub accrued_protocol_fee_usdc: u64,
}
```

The following fields must be immutable after market creation:

```txt
- creator
- mint_fee_bps
- redeem_fee_bps
- creator_share_bps
- protocol_share_bps
```

For v1, `creator_fee_destination` should also be immutable unless a separate governance-approved update rule is explicitly defined.

## 6. Mint Fee Model

Mint fee is charged on user USDC input before reserve composition.

Formula:

```txt
mint_fee_usdc = user_usdc_in * mint_fee_bps / 10000
net_usdc_for_composition = user_usdc_in - mint_fee_usdc
```

Fee split:

```txt
creator_fee_usdc = mint_fee_usdc * creator_share_bps / 10000
protocol_fee_usdc = mint_fee_usdc - creator_fee_usdc
```

With the v1 launch config:

```txt
user_usdc_in = 1000 USDC
mint_fee_bps = 100

mint_fee_usdc = 10 USDC
creator_fee_usdc = 4 USDC
protocol_fee_usdc = 6 USDC
net_usdc_for_composition = 990 USDC
```

Only `net_usdc_for_composition` may be used for reserve composition.

DTF mint amount must be calculated from the actual reserve value added after fee deduction and execution.

```txt
minted_dtf = actual_added_value_usdc / pre_trade_nav
```

The protocol must not mint DTF shares based on gross user USDC input.

## 7. Redeem Fee Model

Axis v1 has no explicit redeem fee.

```txt
redeem_fee_bps = 0
```

Redeem output is based on actual USDC received after reserve unwind.

Formula:

```txt
redeem_fee_usdc = actual_usdc_received * redeem_fee_bps / 10000
user_usdc_out = actual_usdc_received - redeem_fee_usdc
```

For v1:

```txt
redeem_fee_usdc = 0
user_usdc_out = actual_usdc_received
```

However, redeem execution still depends on real market execution.

Therefore, actual USDC received may reflect venue spread, slippage, and price impact.

`min_usdc_out` is required.

```txt
actual_usdc_received >= min_usdc_out
```

If this condition is not satisfied, the redeem transaction must revert.

## 8. Claim-Based Fee Accrual

Axis v1 fees must use claim-based accrual.

Fees must not be immediately transferred to creator or protocol recipients during mint or redeem.

During mint:

```txt
creator_fee_usdc is added to accrued_creator_fee_usdc
protocol_fee_usdc is added to accrued_protocol_fee_usdc
```

During redeem:

```txt
redeem_fee_bps = 0
no explicit redeem fee is accrued in v1
```

Accrued fees must be denominated in USDC or USDC smallest units.

Accrued fees must be claimable through explicit claim or sweep instructions.

Required claim paths:

```txt
ClaimCreatorFee
ClaimProtocolFee
```

or equivalent explicit fee claim instructions.

Accrued fees are not DTF reserves.

Accrued fees must not be included in NAV.

Accrued fees must not be counted as reserve backing.

## 9. Fee Custody

Axis v1 must separate fee custody from reserve custody.

Fee custody may be implemented using a per-market USDC fee vault or an equivalent explicit fee custody account.

The following invariant must hold:

```txt
FeeVault != ReserveAccount
```

Fee vault balances must not be included in DTF reserve value.

Fee vault balances must not be included in NAV.

Fee vault balances must not be used as backing for minted DTF supply.

If fee custody is stored in a shared protocol-level fee vault, the protocol must maintain market-level accounting to prevent cross-market claim errors.

Recommended v1 model:

```txt
per-market USDC fee vault
+
market-level accrued_creator_fee_usdc
+
market-level accrued_protocol_fee_usdc
```

## 10. Accounting Invariants

Fee accounting must preserve DTF reserve and NAV correctness.

Required invariants:

```txt
gross_user_usdc_in != reserve_value_added
net_usdc_for_composition <= gross_user_usdc_in
actual_added_value_usdc is based on reserve balance deltas
minted_dtf is based on actual_added_value_usdc
fee amounts are not counted as reserve value
fee vault balance is not counted as NAV
redeem user output is actual USDC received minus explicit redeem fee
```

For v1:

```txt
redeem_fee_bps = 0
user_usdc_out = actual_usdc_received
```

Creator/protocol fee accounting must never cause over-minting.

Creator/protocol fee accounting must never reduce the reserve value backing existing DTF holders.

## 11. Requirements

### FEE-001: Creator fee must be a required protocol concept

Creator fee must be implemented as a first-class Axis v1 protocol concept.

Acceptance criteria:

```txt
- creator fee is included in the contract account model
- creator fee is included in mint accounting
- creator fee is included in fee claim accounting
- creator fee is included in app integration fields where relevant
- creator fee is included in tests
```

### FEE-002: Each DTF market must store a creator address

Each DTF market must store the creator address that created the market.

Acceptance criteria:

```txt
- creator is stored in DTF market state
- creator is set at market creation
- creator is immutable after market creation
- creator can be used for app display and partner demo context
```

### FEE-003: Each DTF market must store creator fee destination

Each DTF market must store the destination that can receive creator fee claims.

Acceptance criteria:

```txt
- creator_fee_destination is stored
- creator_fee_destination is set at market creation
- creator_fee_destination must be valid
- creator_fee_destination should be immutable for v1
- claim authorization rules are explicit
```

### FEE-004: Creators must not customize fee bps per market

Creators must not be able to set arbitrary fee bps values when creating a DTF market.

Acceptance criteria:

```txt
- mint_fee_bps is derived from protocol config or preset
- redeem_fee_bps is derived from protocol config or preset
- creator_share_bps is derived from protocol config or preset
- protocol_share_bps is derived from protocol config or preset
- invalid custom fee input is rejected or ignored
```

### FEE-005: Market fee configuration must be immutable after creation

Market fee configuration must not be changed after DTF market creation.

Acceptance criteria:

```txt
- mint_fee_bps cannot be changed after market creation
- redeem_fee_bps cannot be changed after market creation
- creator_share_bps cannot be changed after market creation
- protocol_share_bps cannot be changed after market creation
- no market-level SetFee instruction exists for v1 unless explicitly governed by a future spec
```

### FEE-006: Mint fee must be charged on USDC input

Mint fee must be calculated from gross user USDC input.

Acceptance criteria:

```txt
- gross user USDC input is measured
- mint fee is calculated from gross input
- mint_fee_bps = 100 for v1 launch config
- mint fee is separated before reserve composition
- creator fee amount is calculated
- protocol fee amount is calculated
```

### FEE-007: Mint fee must be deducted before reserve composition

Only net USDC after mint fee deduction may be used for reserve composition.

Acceptance criteria:

```txt
- net_usdc_for_composition = user_usdc_in - mint_fee_usdc
- CPI execution uses net USDC allocation
- fee amount is not swapped into reserve assets
- fee amount is not counted as DTF reserve value
```

### FEE-008: Minted DTF amount must be based on net actual reserve value

Minted DTF amount must be calculated from the actual reserve value added after fee deduction and execution.

Acceptance criteria:

```txt
- minted DTF is not based on gross user USDC input
- minted DTF is not based only on quote
- minted DTF is based on actual reserve balance deltas
- fee amounts are not included in actual_added_value_usdc
```

### FEE-009: Redeem must have no explicit Axis exit fee in v1

Axis v1 must not charge an explicit protocol or creator fee on redeem.

Acceptance criteria:

```txt
- redeem_fee_bps = 0
- max_redeem_fee_bps = 0
- no creator fee is charged on redeem
- no protocol fee is charged on redeem
- user_usdc_out = actual_usdc_received
```

### FEE-010: Redeem must still enforce execution protection

Redeem having no explicit exit fee does not remove execution risk controls.

Acceptance criteria:

```txt
- redeem uses actual USDC received after execution
- min_usdc_out is required
- execution spread is separate from protocol fee
- insufficient output causes transaction revert
- failed unwind causes transaction revert
```

### FEE-011: Creator and protocol shares must sum to 10000 bps

Creator and protocol fee shares must fully account for fee distribution.

Acceptance criteria:

```txt
- creator_share_bps = 4000 for v1 launch config
- protocol_share_bps = 6000 for v1 launch config
- creator_share_bps + protocol_share_bps == 10000
- invalid share configuration is rejected
```

### FEE-012: Fee bps must be bounded by protocol-level caps

Protocol-level caps must bound market fee values.

Acceptance criteria:

```txt
- mint_fee_bps <= max_mint_fee_bps
- redeem_fee_bps <= max_redeem_fee_bps
- max_mint_fee_bps = 300 for v1
- max_redeem_fee_bps = 0 for v1
- invalid fee configuration fails
```

### FEE-013: Fees must use claim-based accrual

Axis v1 fees must be accrued and claimed later.

Acceptance criteria:

```txt
- fees are not immediately transferred during mint
- creator fee amount is added to accrued creator fee balance
- protocol fee amount is added to accrued protocol fee balance
- accrued fees are denominated in USDC or USDC smallest units
- accrued fees are not counted as DTF reserves
- accrued fees are not included in NAV
- accrued fees are claimable through explicit instructions
```

### FEE-014: Creator fee claim must be explicit

Creator fee recipient must claim accrued creator fees through an explicit instruction.

Acceptance criteria:

```txt
- creator fee claim instruction exists
- only authorized creator fee destination or defined authority can claim
- claim transfers accrued USDC to the creator fee destination
- claimed amount is subtracted from accrued creator fee balance
- double claim is impossible
- claim failure does not affect reserve accounting
```

### FEE-015: Protocol fee claim must be explicit

Protocol fee must be claimable or sweepable through an explicit protocol instruction.

Acceptance criteria:

```txt
- protocol fee claim or sweep instruction exists
- only protocol treasury authority can claim or sweep
- claim transfers accrued USDC to protocol treasury
- claimed amount is subtracted from accrued protocol fee balance
- double claim is impossible
- claim failure does not affect reserve accounting
```

### FEE-016: Fee custody must be separated from reserve custody

Fee custody must not be mixed with DTF reserve custody.

Acceptance criteria:

```txt
- fee custody account is separate from reserve accounts
- fee vault balance is not included in reserve value
- fee vault balance is not included in NAV
- fee vault balance is not treated as DTF backing
- fee claim does not change reserve balances
```

### FEE-017: Fee accounting must not break reserve or NAV accounting

Fee accounting must be separate from DTF reserve accounting.

Acceptance criteria:

```txt
- fee amounts are not included in actual_added_value_usdc
- fee amounts are not counted as reserve value
- NAV calculation uses reserve balances only
- mint share calculation uses net actual reserve value
- redeem user output uses actual USDC received
- accrued fee balances are excluded from NAV
```

### FEE-018: Fee behavior must be test-covered

Fee behavior must be covered by unit and integration tests.

Acceptance criteria:

```txt
- mint fee calculation test
- creator/protocol split test
- protocol cap test
- redeem zero-fee test
- invalid fee config test
- fee accrual test
- creator claim test
- protocol claim test
- double claim rejection test
- fee accounting with actual balance delta test
- NAV consistency test
```

## 12. Required Test Scenarios

### 12.1 Mint Fee Tests

```txt
- mint with 100 USDC input
- mint with 1000 USDC input
- mint with uneven fee rounding
- mint with exact fee split
- mint with invalid fee config
- mint with max fee cap
- mint where actual reserve value differs from quote
```

### 12.2 Claim Tests

```txt
- creator claims accrued fee
- protocol claims accrued fee
- unauthorized creator claim fails
- unauthorized protocol claim fails
- double creator claim fails
- double protocol claim fails
- claim with zero accrued balance behaves correctly
```

### 12.3 Accounting Tests

```txt
- fee amount is excluded from actual_added_value_usdc
- fee vault is excluded from NAV
- reserve accounts are unchanged by fee claim
- minted DTF amount is based on net actual reserve value
- redeem output has no explicit fee deduction
- accrued creator and protocol fee balances match expected values
```

### 12.4 Failure Tests

```txt
- invalid creator fee destination fails
- invalid protocol treasury fails
- creator_share_bps + protocol_share_bps != 10000 fails
- mint_fee_bps > max_mint_fee_bps fails
- redeem_fee_bps > max_redeem_fee_bps fails
- custom creator fee input fails or is ignored
- post-creation fee update fails
```

## 13. App Integration Notes

The app must be able to display fee-related information where relevant.

Required app-readable fields:

```txt
- creator
- creator_fee_destination
- mint_fee_bps
- redeem_fee_bps
- creator_share_bps
- protocol_share_bps
- accrued_creator_fee_usdc where applicable
- accrued_protocol_fee_usdc where applicable
```

The app must not present execution spread as Axis protocol fee.

The app must distinguish:

```txt
- Axis mint fee
- execution spread
- slippage
- price impact
- estimated output
- actual execution result
```

Detailed UI and UX requirements are out of scope for this document.

## 14. Out of Scope

The following are out of scope for Axis v1 fee model requirements:

```txt
- creator-customizable fee bps
- post-creation fee updates
- explicit redeem exit fee
- rebalance fee
- secondary-market trading fee
- Titan routing fee
- leverage product fee
- fee paid in DTF tokens
- fee paid in underlying reserve assets
```

These may be considered in future versions if needed.

## 15. Open Questions

The following decisions remain open:

```txt
- exact account layout for fee custody
- whether fee custody is per-market or shared with strict accounting
- whether creator_fee_destination is strictly immutable or updateable through governance in a future version
- exact rounding rules for fee calculation
- exact claim instruction names
- whether creator and protocol claim instructions are separate or unified
- whether protocol fee is claimed per market or swept across multiple markets
- event/log format for fee accrual and claims
```

## 16. Issue Candidates

```txt
- Add ProtocolFeeConfig
- Add MarketFeeState
- Store creator in DTF market state
- Store creator fee destination
- Store market fee snapshot at creation
- Enforce immutable market fee config
- Implement USDC-side mint fee calculation
- Implement creator/protocol fee split
- Implement claim-based fee accrual
- Implement creator fee claim instruction
- Implement protocol fee claim instruction
- Separate fee custody from reserve custody
- Exclude fee vault from NAV
- Add mint fee tests
- Add redeem zero-fee tests
- Add creator/protocol split tests
- Add fee cap validation tests
- Add claim authorization tests
- Add double-claim rejection tests
- Add NAV consistency tests
- Add app-readable fee fields
```

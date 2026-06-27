# [P0-IMPL-20] Implement redeem DTF burn path / redeemのDTFバーン経路を実装

## Summary / 概要

Implement the first stage of redeem: validate user DTF input, compute redeem share from pre-redeem supply, compute pro-rata reserve amounts, and burn or otherwise remove redeemed DTF tokens from circulating supply only when the redeem path succeeds.

## Requirement Areas / 関連Requirement Area

- REDEEM-001 to REDEEM-007
- REDEEM-016, REDEEM-017
- PRICE-013
- FEE-009, FEE-010
- PMV-005

## Source Documents / 参照ドキュメント

- `requirements/04-redeem-requirements.md`
- `requirements/06-pricing-nav-requirements.md`
- `requirements/13-fee-model-requirements.md`
- `requirements/12-pre-mainnet-validation-requirements.md`
- `requirements/19-axis-core-implementation-rfc.md`

## Scope / 対象範囲

- Validate user source token account mint equals market DTF mint.
- Validate redeem amount is greater than zero.
- Validate market status allows redeem.
- Support exit-only behavior where mint/creation can be disabled but redeem remains enabled when safe.
- Read total DTF supply before burn or supply mutation.
- Compute redeem share from `dtf_amount_in / total_supply_before`.
- Compute pro-rata reserve amount per asset.
- Validate reserve accounts.
- Validate each asset is redeem-enabled unless emergency policy explicitly defines otherwise.
- Burn or remove redeemed DTF tokens only after successful accounting path.

## Out of Scope / 対象外

- Reserve unwind CPI execution.
- Actual USDC payout.
- Production venue CPI.
- Redeem fee implementation, since v1 explicit redeem fee is zero.
- Direct return of underlying basket assets to user.

## Acceptance Criteria / 完了条件

- Wrong DTF mint fails.
- Non-DTF token input fails.
- Zero redeem amount fails.
- Redeem share uses pre-redeem supply.
- Redeem share cannot exceed 100%.
- Reserve amount is pro-rata to pre-redeem reserve balances.
- Wrong or missing reserve account fails.
- `redeem_enabled=false` blocks normal redeem for that asset.
- Failed redeem does not permanently burn DTF tokens.
- No creator/protocol fee accrues during redeem.

## Test Requirements / 必要なテスト

- Successful redeem input validation test.
- Wrong DTF mint failure test.
- Zero amount failure test.
- Pre-redeem supply calculation test.
- Pro-rata reserve amount calculation test.
- Disabled redeem asset failure test.
- Failed redeem no-permanent-burn test.
- Zero redeem fee/no accrual test.

## Dependencies / 依存関係

- P0-IMPL-14 Token-2022 DTF mint.
- P0-IMPL-19 DTF mint amount calculation from actual reserve value.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `implementation`
- `redeem`
- `token-2022`
- `accounting`

## Suggested Owner / 推奨Owner

Toby primary, ADP review.

## Notes / 補足

Burn ordering must preserve rollback behavior. Do not burn first in a way that can leave users with a permanent burn after failed unwind or min_usdc_out checks.

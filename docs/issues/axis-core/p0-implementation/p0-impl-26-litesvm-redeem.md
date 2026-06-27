# [P0-IMPL-26] Add end-to-end LiteSVM redeem test / LiteSVMのredeem E2Eテストを追加

## Summary / 概要

Add deterministic LiteSVM coverage proving a user can redeem DTF tokens, burn or remove them from circulating supply, unwind reserve assets through the controlled adapter path, and receive actual USDC output.

## Requirement Areas / 関連Requirement Area

- REDEEM-001 to REDEEM-019
- EXEC-013, EXEC-020
- PRICE-013
- FEE-009, FEE-010, FEE-017
- PMV-003, PMV-005, PMV-010, PMV-014
- NFR-REL-004, NFR-OBS-001

## Source Documents / 参照ドキュメント

- `requirements/04-redeem-requirements.md`
- `requirements/05-swap-cpi-execution-requirements.md`
- `requirements/06-pricing-nav-requirements.md`
- `requirements/10-non-functional-requirements.md`
- `requirements/12-pre-mainnet-validation-requirements.md`
- `requirements/13-fee-model-requirements.md`

## Scope / 対象範囲

- Reuse or extend the create_market + mint fixture from P0-IMPL-25.
- Redeem a positive amount of user DTF tokens.
- Compute redeem share from pre-redeem supply.
- Compute pro-rata reserve amount per asset.
- Execute controlled asset -> USDC unwind.
- Verify reserve balances before/after.
- Verify actual USDC received by balance delta.
- Verify `min_usdc_out` enforcement.
- Verify DTF supply/user balance decreases only on successful redeem.
- Verify no creator/protocol fee accrues on redeem.

## Out of Scope / 対象外

- Production Orca/Raydium CPI tests.
- Public Devnet deployment.
- Direct reserve-asset redemption to user.
- Redeem fee.
- Auction/ClearCorrection/JIT tests.

## Acceptance Criteria / 完了条件

- Test starts from a minted reserve-backed market.
- Redeem burns/removes DTF tokens only after successful validation.
- Reserve unwind uses approved controlled path.
- Actual USDC output is measured by balance delta.
- User receives `actual_usdc_received`.
- `min_usdc_out` passes when satisfied and fails when not satisfied.
- Redeem accrues no creator or protocol fee.
- Test is deterministic and suitable for CI.

## Test Requirements / 必要なテスト

- Successful redeem E2E test.
- Pre-redeem supply/redeem share assertion.
- Reserve balance before/after assertion.
- Actual USDC output assertion.
- DTF burn/supply assertion.
- No redeem fee accrual assertion.
- `min_usdc_out` failure assertion may be in this issue or P0-IMPL-27 if grouped with negative tests.

## Dependencies / 依存関係

- P0-IMPL-20 Redeem DTF burn path.
- P0-IMPL-21 Reserve unwind and USDC payout path.
- P0-IMPL-25 Create_market + mint E2E test.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `tests`
- `litesvm`
- `redeem`
- `e2e`

## Suggested Owner / 推奨Owner

Toby.

## Notes / 補足

Redeem tests should explicitly distinguish execution spread/slippage from protocol fees.

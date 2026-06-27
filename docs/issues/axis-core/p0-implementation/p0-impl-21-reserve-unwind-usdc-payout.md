# [P0-IMPL-21] Implement reserve unwind and USDC payout path / リザーブ巻き戻しとUSDC支払い経路を実装

## Summary / 概要

Implement reserve unwind through the controlled execution path and pay the user actual USDC received after validating `min_usdc_out`.

Redeem output must be based on actual USDC balance delta, not quote or backend estimate.

## Requirement Areas / 関連Requirement Area

- REDEEM-008 to REDEEM-019
- EXEC-001 to EXEC-014
- EXEC-019, EXEC-020
- PRICE-013
- FEE-009, FEE-010
- PMV-005
- APPIF-008, APPIF-009, APPIF-015, APPIF-016

## Source Documents / 参照ドキュメント

- `requirements/04-redeem-requirements.md`
- `requirements/05-swap-cpi-execution-requirements.md`
- `requirements/06-pricing-nav-requirements.md`
- `requirements/13-fee-model-requirements.md`
- `requirements/15-app-contract-interface-requirements.md`
- `requirements/12-pre-mainnet-validation-requirements.md`

## Scope / 対象範囲

- Validate approved unwind route per reserve asset.
- Execute reserve asset -> USDC through controlled adapter path for P0.
- Enforce route direction, input mint, output mint, venue/adapter identity, and route enabled state.
- Measure actual USDC received by pre/post USDC balance delta.
- Enforce `actual_usdc_received >= min_usdc_out`.
- Pay user `user_usdc_out = actual_usdc_received` for v1.
- Ensure reserve transfers do not exceed computed pro-rata redeem amounts.
- Preserve all-or-nothing redeem behavior.
- Emit/log useful redeem result fields where supported.

## Out of Scope / 対象外

- Explicit Axis redeem fee.
- Production Orca/Raydium CPI readiness.
- Auction/ClearCorrection unwind logic.
- Returning reserve basket assets directly to user.
- Treating execution spread as protocol fee.

## Acceptance Criteria / 完了条件

- Approved unwind route succeeds with controlled adapter.
- Missing route fails.
- Disabled route fails.
- Wrong venue, pool/account, direction, input mint, or output mint fails.
- `min_usdc_out` is required and enforced against actual USDC delta.
- Actual USDC received below `min_usdc_out` reverts full transaction.
- User payout equals actual USDC received for v1.
- No creator/protocol fee accrues on redeem.
- Failed redeem does not transfer USDC, drain reserves, or permanently burn DTF.

## Test Requirements / 必要なテスト

- Successful controlled-adapter redeem/unwind test.
- Actual USDC delta test.
- `min_usdc_out` failure test.
- Missing/disabled route failure tests.
- Wrong input/output mint failure tests.
- Pro-rata source-drain bounds test.
- Failed unwind rollback test.
- Execution spread not fee test.

## Dependencies / 依存関係

- P0-IMPL-17 Controlled adapter interface.
- P0-IMPL-18 Balance delta validation.
- P0-IMPL-20 Redeem DTF burn path.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `implementation`
- `redeem`
- `cpi`
- `usdc-payout`

## Suggested Owner / 推奨Owner

Toby.

## Notes / 補足

This issue proves the P0 redeem lifecycle in a controlled environment only. It is not proof of production venue readiness.

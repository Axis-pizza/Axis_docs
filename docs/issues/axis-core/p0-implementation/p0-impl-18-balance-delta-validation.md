# [P0-IMPL-18] Implement pre/post reserve balance delta validation / 実行前後のリザーブ残高差分検証を実装

## Summary / 概要

Implement reusable pre/post token balance delta validation for mint and redeem accounting.

Actual balance deltas are protocol truth for P0 accounting. Quotes, backend estimates, target weights, and route plan outputs must not be used as final accounting truth.

## Requirement Areas / 関連Requirement Area

- MINT-009, MINT-010, MINT-011
- REDEEM-009, REDEEM-010, REDEEM-011, REDEEM-017
- EXEC-007, EXEC-008
- PRICE-001, PRICE-012, PRICE-013
- PMV-003
- NFR-SEC-002, NFR-REL-004

## Source Documents / 参照ドキュメント

- `requirements/03-mint-requirements.md`
- `requirements/04-redeem-requirements.md`
- `requirements/05-swap-cpi-execution-requirements.md`
- `requirements/06-pricing-nav-requirements.md`
- `requirements/10-non-functional-requirements.md`
- `requirements/12-pre-mainnet-validation-requirements.md`

## Scope / 対象範囲

- Capture pre-execution reserve token balances for mint.
- Capture post-execution reserve token balances for mint.
- Compute actual received reserve asset delta.
- Capture pre/post USDC balances for redeem.
- Compute actual USDC received.
- Enforce `actual_received_i >= min_out_i`.
- Enforce `actual_usdc_received >= min_usdc_out`.
- Validate output balance delta is positive and input balance movement is in expected direction where applicable.
- Return actual deltas to mint/redeem accounting.

## Out of Scope / 対象外

- Pricing/NAV calculation.
- DTF mint amount calculation.
- Production venue account validation.
- Fee claim logic.
- External pool price display.

## Acceptance Criteria / 完了条件

- Mint accounting records pre/post reserve balances.
- Mint actual received amount is `post_reserve_balance - pre_reserve_balance`.
- Redeem accounting records pre/post USDC balances.
- Redeem actual received amount is `post_usdc_balance - pre_usdc_balance`.
- Quote output cannot satisfy accounting checks.
- Target weights cannot be used as actual reserve balances.
- `min_out` and `min_usdc_out` are enforced against actual deltas.
- Delta failure reverts parent transaction.

## Test Requirements / 必要なテスト

- Mint actual reserve delta success test.
- Redeem actual USDC delta success test.
- Quote mismatch does not over-credit test.
- `min_out` below actual passes/equal passes/above actual fails test.
- Wrong reserve account delta failure test.
- Rollback test for failed delta validation.

## Dependencies / 依存関係

- P0-IMPL-13 Reserve vault PDAs.
- P0-IMPL-17 Controlled adapter interface.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `implementation`
- `accounting`
- `balance-delta`
- `security`

## Suggested Owner / 推奨Owner

Toby.

## Notes / 補足

This is one of the highest-risk safety surfaces. Keep it small, reusable, and heavily tested before layering mint/redeem value calculations on top.

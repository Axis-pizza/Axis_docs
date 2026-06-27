# [P0-IMPL-16] Implement mint instruction USDC intake and fee separation / USDCミント入力とフィー分離を実装

## Summary / 概要

Implement the first stage of the mint instruction: validate USDC input, calculate mint fee, separate fee custody from reserves, and compute net USDC allocations for reserve composition.

Mint accounting must use USDC-side entry and must not compose fees into reserves.

## Requirement Areas / 関連Requirement Area

- MINT-001 to MINT-007
- MINT-017, MINT-018
- FEE-006 to FEE-008
- FEE-013, FEE-016, FEE-017
- POLICY-006
- APPIF-006, APPIF-007, APPIF-011

## Source Documents / 参照ドキュメント

- `requirements/03-mint-requirements.md`
- `requirements/07-execution-policy-risk-controls.md`
- `requirements/13-fee-model-requirements.md`
- `requirements/15-app-contract-interface-requirements.md`
- `requirements/19-axis-core-implementation-rfc.md`

## Scope / 対象範囲

- Validate user source token account mint equals configured USDC mint.
- Validate market status allows mint.
- Calculate mint fee from gross USDC input.
- Split mint fee into creator and protocol amounts using market fee config.
- Transfer or retain fee amount in fee custody according to accepted fee design.
- Compute `net_usdc_for_composition`.
- Compute per-asset USDC allocations from net USDC and target weights.
- Enforce hard minimum allocation of 1 USDC.
- Enforce asset `max_trade_usdc`.
- Enforce each asset `mint_enabled=true`.
- Ensure failed mint does not accrue fees or mint DTF tokens.

## Out of Scope / 対象外

- Actual CPI swap execution.
- Actual reserve balance delta validation.
- Final DTF mint amount calculation.
- Redeem behavior.
- Fee claim instructions.
- Production venue readiness.

## Acceptance Criteria / 完了条件

- Non-USDC input fails.
- Paused or deprecated market blocks mint according to market status rules.
- 1000 USDC input with 1% mint fee produces 10 USDC fee and 990 USDC net composition value.
- Creator/protocol split follows immutable market fee config.
- Per-asset allocation uses net USDC, not gross USDC.
- Allocation below 1 USDC fails.
- Allocation above asset max trade fails.
- Disabled mint asset fails.
- Fee custody is distinct from reserve custody.
- Fee amount is not counted as reserve value or NAV backing.

## Test Requirements / 必要なテスト

- USDC input validation test.
- Non-USDC failure test.
- Mint fee calculation test.
- Creator/protocol split test.
- Net allocation test.
- Minimum allocation failure test.
- Max trade failure test.
- Disabled mint asset failure test.
- Failed mint no-fee-accrual test.

## Dependencies / 依存関係

- P0-IMPL-15 Create market.
- P0-IMPL-22 Fee accrual state, if fee state is split into a separate implementation issue.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `implementation`
- `mint`
- `fees`
- `accounting`

## Suggested Owner / 推奨Owner

Toby primary, ADP review.

## Notes / 補足

This issue should not determine final minted DTF amount. That belongs to P0-IMPL-19 after actual reserve balance deltas and pricing/NAV checks are available.

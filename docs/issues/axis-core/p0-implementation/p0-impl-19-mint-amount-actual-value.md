# [P0-IMPL-19] Implement DTF mint amount calculation from actual reserve value / 実リザーブ価値に基づくDTFミント量計算を実装

## Summary / 概要

Implement DTF mint amount calculation based on actual reserve value added after fee deduction and controlled execution.

Minted DTF amount must be derived from actual reserve deltas and pre-trade NAV, not gross USDC input, net USDC estimate, target weights, backend quote, or route-plan estimate.

## Requirement Areas / 関連Requirement Area

- MINT-011 to MINT-015
- PRICE-001, PRICE-002, PRICE-010, PRICE-012
- FEE-008, FEE-017
- APPIF-007
- PMV-004

## Source Documents / 参照ドキュメント

- `requirements/03-mint-requirements.md`
- `requirements/06-pricing-nav-requirements.md`
- `requirements/13-fee-model-requirements.md`
- `requirements/15-app-contract-interface-requirements.md`
- `requirements/19-axis-core-implementation-rfc.md`

## Scope / 対象範囲

- Snapshot pre-trade reserve balances before mint execution.
- Compute or load pre-trade NAV before reserve mutation.
- Support initial NAV = 1 USDC when DTF supply is zero.
- Use actual reserve balance deltas from P0-IMPL-18.
- Convert actual received assets into `actual_added_value_usdc` using the approved P0 pricing source path.
- Calculate minted DTF amount from `actual_added_value_usdc / pre_trade_nav`.
- Exclude mint fee and fee custody from actual added value.
- Mint DTF tokens only after all validation and accounting checks pass.

## Out of Scope / 対象外

- Full production PricingSourceRegistry if P0 uses a constrained manual/fixed price path.
- Production oracle integrations.
- Redeem USDC payout.
- Secondary-market pool price handling.
- Final mainnet rounding/dust policy if it remains an engineering-review blocker.

## Acceptance Criteria / 完了条件

- First mint uses initial NAV = 1 USDC when supply is zero.
- Subsequent mint uses pre-trade NAV computed before reserve mutation.
- Minted DTF amount changes with actual execution result.
- Optimistic quote cannot over-mint DTF.
- Gross USDC input is not used directly as minted value.
- Net USDC allocation is not used directly as minted value.
- Mint fees are excluded from `actual_added_value_usdc`.
- Fee vault balances are excluded from NAV.
- Missing/stale/disabled pricing source fails where the P0 pricing path requires it.

## Test Requirements / 必要なテスト

- Initial mint NAV = 1 test.
- Subsequent mint pre-trade NAV test.
- Actual delta drives minted amount test.
- Optimistic quote cannot over-mint test.
- Fee exclusion from NAV and actual added value test.
- Missing/disabled pricing source failure test if pricing validation is implemented in P0.
- Rounding/dust deterministic test once math rules are accepted.

## Dependencies / 依存関係

- P0-IMPL-16 Mint USDC intake and fee separation.
- P0-IMPL-18 Balance delta validation.
- P0-IMPL-22 Fee accrual state.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `implementation`
- `mint`
- `nav`
- `pricing`
- `accounting`

## Suggested Owner / 推奨Owner

ADP design review, Toby implementation.

## Notes / 補足

Exact fixed-point representation, decimals normalization, rounding, and dust handling should be explicitly reviewed before this issue is considered complete.

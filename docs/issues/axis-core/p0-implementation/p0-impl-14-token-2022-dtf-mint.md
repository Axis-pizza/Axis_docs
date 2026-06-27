# [P0-IMPL-14] Wire Token-2022 DTF mint authority and supply validation / Token-2022 DTFミント権限と供給量検証を接続

## Summary / 概要

Wire Axis Core to create or validate the Token-2022 DTF mint for each market and control mint/burn authority through the Axis program authority model.

The exact Token-2022 extension set may remain an engineering-review item, but P0 must prove DTF mint and burn behavior with supply validation.

## Requirement Areas / 関連Requirement Area

- DTF-011
- MINT-011, MINT-012
- REDEEM-001, REDEEM-004, REDEEM-016
- NFR-SEC-005
- PMV-002, PMV-005

## Source Documents / 参照ドキュメント

- `requirements/02-dtf-market-requirements.md`
- `requirements/03-mint-requirements.md`
- `requirements/04-redeem-requirements.md`
- `requirements/10-non-functional-requirements.md`
- `requirements/12-pre-mainnet-validation-requirements.md`
- `requirements/19-axis-core-implementation-rfc.md`

## Scope / 対象範囲

- Validate or initialize the market DTF Token-2022 mint, per the accepted instruction spec.
- Ensure DTF mint authority is Axis-controlled.
- Ensure burn path can remove redeemed DTF tokens from circulating supply.
- Validate mint supply when calculating initial NAV or redeem share.
- Provide test helpers for Token-2022 mint/burn in LiteSVM.

## Out of Scope / 対象外

- Final metadata extension choice if unresolved.
- Secondary-market pool creation.
- Token-2022 StockToken/xStocks extension support.
- Legacy SPL-only mint assumptions.

## Acceptance Criteria / 完了条件

- Each market has exactly one DTF mint.
- DTF mint authority is Axis-controlled.
- Wrong DTF mint fails market/mint/redeem validation.
- Initial supply zero path can use initial NAV = 1 USDC.
- Mint path can mint DTF tokens to user destination after accounting succeeds.
- Redeem path can burn DTF tokens or otherwise remove redeemed amount from circulating supply after accounting succeeds.
- Failed mint does not increase DTF supply.
- Failed redeem does not permanently burn DTF supply.

## Test Requirements / 必要なテスト

- Token-2022 mint authority validation test.
- Successful DTF mint test.
- Successful DTF burn test.
- Wrong DTF mint failure test.
- Failed mint no-supply-change test.
- Failed redeem no-permanent-burn test.

## Dependencies / 依存関係

- Existing `P0-TOKEN-03` Token-2022 DTF mint/burn spike.
- P0-IMPL-12 DTFMarket account.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `implementation`
- `token-2022`
- `dtf-mint`

## Suggested Owner / 推奨Owner

Toby primary, ADP review.

## Notes / 補足

Do not rely on an internal total_supply mirror as protocol truth if Token-2022 mint supply can be validated directly.

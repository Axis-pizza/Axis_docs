# [P0-IMPL-15] Implement create_market instruction / create_market命令を実装

## Summary / 概要

Implement `create_market` for a reserve-backed DTF market.

This instruction should validate market assets, weights, creator fee destination, reserve vaults, and DTF mint wiring without introducing legacy direct basket deposit/withdraw behavior.

## Requirement Areas / 関連Requirement Area

- DTF-001 to DTF-017
- FEE-001 to FEE-005
- APPIF-005
- PMV-002
- NFR-SEC-001, NFR-SEC-004, NFR-SEC-005

## Source Documents / 参照ドキュメント

- `requirements/02-dtf-market-requirements.md`
- `requirements/08-asset-universe-requirements.md`
- `requirements/13-fee-model-requirements.md`
- `requirements/15-app-contract-interface-requirements.md`
- `requirements/12-pre-mainnet-validation-requirements.md`

## Scope / 対象範囲

- Implement `create_market` instruction.
- Validate asset count is 2 to 5.
- Reject duplicate asset mints.
- Enforce total weights equal 10000 bps.
- Enforce each weight is at least 100 bps.
- Enforce each weight is within asset `max_weight_bps`.
- Enforce each asset exists in the AssetRegistry and `creation_enabled=true`.
- Initialize or validate reserve vaults for each asset.
- Initialize or validate the DTF Token-2022 mint.
- Store creator and creator fee destination.
- Derive immutable market fee config from ProtocolConfig.
- Set initial market status according to the accepted instruction spec.

## Out of Scope / 対象外

- Minting DTF tokens.
- Redeeming DTF tokens.
- Route execution.
- Production venue CPI.
- Off-chain market metadata registry.
- Market-level TVL cap.

## Acceptance Criteria / 完了条件

- 0 or 1 asset market creation fails.
- 2 asset market creation succeeds when all checks pass.
- 5 asset market creation succeeds when all checks pass.
- 6 asset market creation fails.
- Duplicate asset market fails.
- Invalid weight sum fails.
- Weight below 100 bps fails.
- Weight above asset max fails.
- Unknown asset fails.
- `creation_enabled=false` asset fails.
- Reserve vault and DTF mint references are stored or derivable.
- Creator cannot customize fee bps during market creation.
- Market fee config is derived from ProtocolConfig and immutable.

## Test Requirements / 必要なテスト

- Market asset count tests.
- Duplicate asset test.
- Weight sum and min weight tests.
- Asset max weight test.
- Unknown/disabled asset tests.
- Creator fee destination validation test.
- Derived fee config test.
- Reserve vault and DTF mint validation tests.

## Dependencies / 依存関係

- P0-IMPL-10 ProtocolConfig.
- P0-IMPL-11 Asset Registry.
- P0-IMPL-12 DTFMarket account.
- P0-IMPL-13 Reserve vault PDAs.
- P0-IMPL-14 Token-2022 DTF mint.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `implementation`
- `create-market`
- `dtf-market`

## Suggested Owner / 推奨Owner

Toby primary, ADP review.

## Notes / 補足

This instruction creates the protocol market state. It should not create or depend on secondary-market pools, auction accounts, Titan integration, or frontend/backend metadata truth.

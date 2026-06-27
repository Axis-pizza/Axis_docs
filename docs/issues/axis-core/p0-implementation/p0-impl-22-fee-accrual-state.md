# [P0-IMPL-22] Implement creator/protocol fee accrual state / クリエイター・プロトコルフィーの発生状態を実装

## Summary / 概要

Implement creator/protocol fee accrual state for Axis Core.

Creator fees and protocol fees are first-class v1 protocol concepts. They must accrue on mint, remain claim-based, and stay separate from reserve custody and NAV.

## Requirement Areas / 関連Requirement Area

- DTF-013 to DTF-017
- MINT-003, MINT-017, MINT-018
- REDEEM-012, REDEEM-018
- FEE-001 to FEE-013
- FEE-016 to FEE-018
- ADMIN-008
- PMV-006

## Source Documents / 参照ドキュメント

- `requirements/02-dtf-market-requirements.md`
- `requirements/03-mint-requirements.md`
- `requirements/04-redeem-requirements.md`
- `requirements/13-fee-model-requirements.md`
- `requirements/09-admin-safety-requirements.md`
- `requirements/12-pre-mainnet-validation-requirements.md`

## Scope / 対象範囲

- Define fee accrual state location according to accepted fee model.
- Track `accrued_creator_fee_usdc`.
- Track `accrued_protocol_fee_usdc`.
- Ensure accrued fees are denominated in USDC smallest units or the approved representation.
- Ensure mint fee accrues only after full parent mint succeeds.
- Ensure failed mint does not accrue fees.
- Ensure redeem does not accrue creator/protocol fees in v1.
- Ensure fee vault/custody is separate from reserve vaults.
- Ensure accrued fees are excluded from NAV and DTF backing.

## Out of Scope / 対象外

- Fee claim transfer instructions.
- Changing fee config after market creation.
- Redeem fee.
- Auction revenue accounting.
- DTF-denominated treasury fees.

## Acceptance Criteria / 完了条件

- Creator fee is stored as a required protocol concept.
- Protocol fee is stored as a required protocol concept.
- Mint fee accrues creator/protocol shares based on immutable market fee config.
- Creator and protocol shares sum to 10000 bps.
- Failed mint accrues no fee.
- Redeem accrues no creator/protocol fee.
- Fee balances are readable separately from reserve balances.
- Fee vault balance is excluded from reserve value and NAV.
- Fee claim cannot reduce reserve balances.

## Test Requirements / 必要なテスト

- Mint fee accrual test.
- Creator/protocol split test.
- Invalid fee config test.
- Failed mint no-accrual test.
- Redeem zero-fee/no-accrual test.
- Fee vault excluded from NAV test.
- Fee custody separate from reserve custody test.

## Dependencies / 依存関係

- Existing `P0-FEE-07` fee accounting proposal.
- P0-IMPL-10 ProtocolConfig.
- P0-IMPL-12 DTFMarket account.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `implementation`
- `fees`
- `accounting`

## Suggested Owner / 推奨Owner

ADP design review, Toby implementation.

## Notes / 補足

Do not count fees as reserves or NAV backing. Do not introduce DTF-denominated treasury fees for P0.

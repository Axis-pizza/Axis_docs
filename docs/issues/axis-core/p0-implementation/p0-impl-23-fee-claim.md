# [P0-IMPL-23] Implement fee claim instruction / フィー請求命令を実装

## Summary / 概要

Implement explicit creator and protocol fee claim paths.

Fees are claim-based, not immediately transferred to final recipients during mint. Claiming must not touch reserve vaults, affect NAV, or alter DTF backing.

## Requirement Areas / 関連Requirement Area

- DTF-016, DTF-017
- FEE-013 to FEE-018
- APPIF-010, APPIF-011
- PMV-006
- ADMIN-008

## Source Documents / 参照ドキュメント

- `requirements/02-dtf-market-requirements.md`
- `requirements/13-fee-model-requirements.md`
- `requirements/15-app-contract-interface-requirements.md`
- `requirements/12-pre-mainnet-validation-requirements.md`
- `requirements/09-admin-safety-requirements.md`

## Scope / 対象範囲

- Implement creator fee claim instruction or approved equivalent.
- Implement protocol fee claim/sweep instruction or approved equivalent.
- Validate creator fee destination authorization.
- Validate protocol treasury authority.
- Transfer claimable USDC from fee custody to authorized destination.
- Subtract claimed amount from accrued fee state.
- Prevent double claim.
- Ensure fee claim cannot reduce reserve balances.
- Emit/log useful claim result data where supported.

## Out of Scope / 対象外

- Mint fee accrual implementation.
- Reserve vault transfers.
- Auction revenue claims.
- Creator-custom fee bps.
- Redeem fee claims.

## Acceptance Criteria / 完了条件

- Creator fee can be claimed only by authorized creator destination or defined authority.
- Protocol fee can be claimed only by protocol treasury authority.
- Claim amount is derived from accrued fee state, not user input truth.
- Claimed amount is subtracted from accrued fee state.
- Double claim is impossible.
- Claim failure does not affect reserve accounting.
- Fee claim does not affect NAV.
- Unauthorized claim fails.

## Test Requirements / 必要なテスト

- Successful creator claim test.
- Successful protocol claim test.
- Unauthorized creator claim failure test.
- Unauthorized protocol claim failure test.
- Double claim rejection test.
- Claim does not change reserve balances test.
- Claim does not change NAV/reserve value test.

## Dependencies / 依存関係

- P0-IMPL-22 Fee accrual state.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `implementation`
- `fees`
- `claim`

## Suggested Owner / 推奨Owner

Toby primary, ADP review.

## Notes / 補足

Exact instruction names can follow the accepted instruction surface proposal, but the issue should cover both creator and protocol claim paths.

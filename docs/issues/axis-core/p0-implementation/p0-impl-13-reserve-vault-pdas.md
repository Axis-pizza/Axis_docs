# [P0-IMPL-13] Implement reserve vault PDA creation and validation / リザーブVault PDA作成と検証を実装

## Summary / 概要

Implement reserve vault PDA creation and validation for each DTF market asset.

Reserve vaults are the on-chain custody boundary for DTF backing. They must be program-controlled and must not be confused with fee vaults, user token accounts, venue vaults, external pools, or legacy vault wrappers.

## Requirement Areas / 関連Requirement Area

- DTF-010
- PRICE-001
- MINT-010, MINT-018
- REDEEM-006, REDEEM-017, REDEEM-018
- FEE-016, FEE-017
- NFR-SEC-004

## Source Documents / 参照ドキュメント

- `requirements/02-dtf-market-requirements.md`
- `requirements/03-mint-requirements.md`
- `requirements/04-redeem-requirements.md`
- `requirements/06-pricing-nav-requirements.md`
- `requirements/10-non-functional-requirements.md`
- `requirements/19-axis-core-implementation-rfc.md`

## Scope / 対象範囲

- Define reserve vault PDA derivation per market and asset.
- Create or validate token accounts for each reserve asset.
- Verify reserve vault mint matches expected asset mint.
- Verify reserve vault authority/owner is Axis-controlled.
- Expose helper validation for mint, redeem, NAV, and tests.
- Keep fee custody and reserve custody distinct.

## Out of Scope / 対象外

- Fee vault creation.
- Venue adapter vault validation.
- Public DTF/USDC pool indexing.
- Direct user withdrawal of reserve basket assets.
- Production venue account validation.

## Acceptance Criteria / 完了条件

- Reserve vault PDA is deterministic for each market and asset.
- Wrong reserve account fails validation.
- Reserve vault with wrong mint fails validation.
- Reserve vault with wrong authority fails validation.
- Missing reserve vault fails where required.
- Reserve vault balance can be read for accounting and demo evidence.
- Fee vault is not accepted as reserve vault.

## Test Requirements / 必要なテスト

- Reserve vault derivation test.
- Successful reserve vault creation/validation test.
- Wrong vault failure test.
- Wrong mint failure test.
- Wrong authority failure test.
- Fee vault not accepted as reserve vault test.

## Dependencies / 依存関係

- P0-IMPL-12 DTFMarket account.
- Existing legacy pattern extraction may inform PDA-owned custody safety only.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `implementation`
- `reserves`
- `pda`
- `security`

## Suggested Owner / 推奨Owner

Toby.

## Notes / 補足

Legacy PDA custody patterns may inform safety checks, but the legacy product model must not be ported.

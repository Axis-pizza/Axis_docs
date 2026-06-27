# [P0-IMPL-25] Add end-to-end LiteSVM create_market + mint test / LiteSVMのcreate_market + mint E2Eテストを追加

## Summary / 概要

Add deterministic LiteSVM coverage proving a user can create a DTF market and mint reserve-backed DTF tokens with USDC through the controlled adapter path.

The test must show actual reserve balance deltas, Token-2022 DTF mint evidence, and fee separation.

## Requirement Areas / 関連Requirement Area

- DTF-001 to DTF-017
- MINT-001 to MINT-019
- EXEC-013, EXEC-020
- PRICE-001, PRICE-002, PRICE-012
- FEE-006 to FEE-018
- PMV-002, PMV-003, PMV-004, PMV-010, PMV-014
- NFR-REL-004, NFR-OBS-001

## Source Documents / 参照ドキュメント

- `requirements/02-dtf-market-requirements.md`
- `requirements/03-mint-requirements.md`
- `requirements/05-swap-cpi-execution-requirements.md`
- `requirements/06-pricing-nav-requirements.md`
- `requirements/10-non-functional-requirements.md`
- `requirements/12-pre-mainnet-validation-requirements.md`
- `requirements/13-fee-model-requirements.md`

## Scope / 対象範囲

- Add LiteSVM fixture for ProtocolConfig, assets, routes, test mints, test token accounts, reserve vaults, fee custody, and controlled adapter.
- Create a 2-asset DTF market.
- Mint DTF with USDC.
- Execute controlled USDC -> asset movements using real test tokens.
- Record reserve vault balances before and after.
- Verify actual reserve balance deltas.
- Verify DTF Token-2022 supply/user balance increased by calculated amount.
- Verify fee custody and accrued fee state are separate from reserve/NAV.
- Verify useful logs/events or refreshed state can be used as demo evidence.

## Out of Scope / 対象外

- Production Orca/Raydium CPI tests.
- Public Devnet deployment.
- Titan integration.
- Redeem test, covered by P0-IMPL-26.
- Auction/ClearCorrection/JIT tests.

## Acceptance Criteria / 完了条件

- Test creates a valid 2-asset DTF market.
- Test mints with USDC input.
- Controlled adapter moves real test tokens.
- Reserve vault balances increase by actual received assets.
- DTF amount minted is based on actual reserve value.
- Fee amount is excluded from reserves and NAV.
- Creator/protocol fee accrual is verified.
- Test is deterministic and runs in CI once the LiteSVM scaffold supports it.

## Test Requirements / 必要なテスト

- This issue is itself a test implementation.
- Include assertions for reserve balances before/after.
- Include assertions for DTF mint supply/user balance.
- Include assertions for fee custody and accrued fee state.
- Include assertions that quote/estimate values are not protocol truth where the test fixture can show mismatch.

## Dependencies / 依存関係

- Existing `P0-TEST-02` LiteSVM scaffold.
- P0-IMPL-15 Create market.
- P0-IMPL-16 Mint USDC intake and fee separation.
- P0-IMPL-17 Controlled adapter.
- P0-IMPL-18 Balance delta validation.
- P0-IMPL-19 DTF mint amount calculation.
- P0-IMPL-22 Fee accrual state.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `tests`
- `litesvm`
- `mint`
- `e2e`

## Suggested Owner / 推奨Owner

Toby.

## Notes / 補足

This test provides P0 Core evidence only. It must not be described as production venue readiness or mainnet readiness.

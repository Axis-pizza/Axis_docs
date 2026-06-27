# [P0-IMPL-27] Add negative tests for wrong vault / wrong route / unauthorized signer / 不正Vault・不正ルート・未認可署名者のネガティブテストを追加

## Summary / 概要

Add deterministic negative tests for the most important P0 safety failures.

The goal is to prove invalid accounts, invalid routes, disabled controls, and unauthorized signers fail without partial reserve movement, DTF mint/burn mutation, or fee accrual.

## Requirement Areas / 関連Requirement Area

- DTF-001 to DTF-008
- MINT-008 to MINT-018
- REDEEM-006 to REDEEM-018
- EXEC-001 to EXEC-012, EXEC-020
- ADMIN-002 to ADMIN-008
- PMV-008, PMV-009
- NFR-SEC-001 to NFR-SEC-005
- NFR-REL-001 to NFR-REL-004

## Source Documents / 参照ドキュメント

- `requirements/02-dtf-market-requirements.md`
- `requirements/03-mint-requirements.md`
- `requirements/04-redeem-requirements.md`
- `requirements/05-swap-cpi-execution-requirements.md`
- `requirements/09-admin-safety-requirements.md`
- `requirements/10-non-functional-requirements.md`
- `requirements/12-pre-mainnet-validation-requirements.md`

## Scope / 対象範囲

- Add negative tests for wrong reserve vault.
- Add negative tests for wrong DTF mint.
- Add negative tests for wrong USDC mint.
- Add negative tests for missing/disabled/wrong route.
- Add negative tests for wrong venue/adapter account.
- Add negative tests for unauthorized protocol, asset, route, fee, and pause authority actions.
- Add negative tests for disabled asset creation/mint/redeem.
- Add negative tests for `min_out` and `min_usdc_out` failure.
- Add rollback assertions:
  - failed mint mints no DTF
  - failed mint accrues no fees
  - failed mint does not accept reserve deltas as successful accounting
  - failed redeem does not permanently burn DTF
  - failed redeem does not transfer USDC to user
  - failed redeem does not drain reserves

## Out of Scope / 対象外

- Production venue invalid account tests for Orca/Raydium.
- Fuzzing or property tests unless already available.
- Formal verification.
- Auction/ClearCorrection adversarial tests.

## Acceptance Criteria / 完了条件

- Wrong vault fails.
- Wrong route fails.
- Unauthorized signer fails.
- Disabled route fails.
- Disabled asset action fails according to flag.
- Failed mint/redeem roll back fully.
- Failure tests assert no unwanted DTF supply, reserve, fee, or USDC changes.
- Negative tests run deterministically.

## Test Requirements / 必要なテスト

- This issue is itself a test implementation.
- Group tests by validation area where possible:
  - account validation
  - route validation
  - authority validation
  - pause/disable controls
  - min_out/min_usdc_out
  - rollback invariants

## Dependencies / 依存関係

- P0-IMPL-24 Pause/disable controls.
- P0-IMPL-25 Create_market + mint E2E test.
- P0-IMPL-26 Redeem E2E test.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `tests`
- `litesvm`
- `negative-tests`
- `security`

## Suggested Owner / 推奨Owner

Toby.

## Notes / 補足

Prioritize tests that prevent silent accounting corruption or custody loss. Production venue spoofing tests belong to the production venue readiness track after P0 Core.

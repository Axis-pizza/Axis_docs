# [P0-IMPL-17] Implement controlled adapter execution interface / 制御されたアダプター実行インターフェースを実装

## Summary / 概要

Implement a controlled adapter execution interface for deterministic local tests and P0 accounting validation.

The adapter should move real SPL tokens in a controlled environment and return actual output through balance delta validation. It must be explicitly labeled test/local validation only and must not be treated as production venue readiness.

## Requirement Areas / 関連Requirement Area

- EXEC-001 to EXEC-014
- EXEC-019, EXEC-020
- VENUE-005, VENUE-006
- POLICY-012
- APPIF-003, APPIF-004, APPIF-015
- PMV-003, PMV-004, PMV-005

## Source Documents / 参照ドキュメント

- `requirements/05-swap-cpi-execution-requirements.md`
- `requirements/07-execution-policy-risk-controls.md`
- `requirements/12-pre-mainnet-validation-requirements.md`
- `requirements/14-production-venue-integration-requirements.md`
- `requirements/15-app-contract-interface-requirements.md`
- `requirements/19-axis-core-implementation-rfc.md`

## Scope / 対象範囲

- Define adapter interface shape for P0.
- Implement controlled adapter support for local deterministic tests.
- Validate route reference maps to enabled ApprovedRoute or equivalent P0 route state.
- Validate execution direction:
  - mint: USDC -> reserve asset
  - redeem: reserve asset -> USDC
- Validate input and output mint.
- Validate controlled adapter program/account identity.
- Require and pass through `min_out` or `min_usdc_out`.
- Ensure adapter failure reverts parent mint/redeem.
- Keep route complexity bounded to one route per asset for P0.

## Out of Scope / 対象外

- Orca Whirlpool production CPI.
- Raydium CPMM production CPI.
- Jupiter opaque route bytes.
- Split routing.
- Arbitrary multi-hop route graphs.
- SOL intermediate routes unless a later approved requirement adds them.
- Auction/ClearCorrection settlement paths.

## Acceptance Criteria / 完了条件

- Controlled adapter can move real test SPL tokens.
- Missing route fails.
- Disabled route fails.
- Wrong venue/adapter fails.
- Wrong input mint fails.
- Wrong output mint fails.
- Wrong direction fails.
- Missing `min_out` fails unless explicitly allowed for a test-only case.
- Actual output below minimum fails and rolls back parent transaction.
- Controlled adapter tests are documented as non-production readiness.

## Test Requirements / 必要なテスト

- Controlled adapter successful mint-leg test.
- Controlled adapter successful redeem-leg test.
- Wrong route/venue/input/output/direction failure tests.
- `min_out` failure test.
- Adapter failure rollback test.
- Test assertion or documentation that controlled adapter does not satisfy production venue readiness.

## Dependencies / 依存関係

- Existing `P0-ROUTE-06` ApprovedRoute model proposal.
- Existing `P0-CPI-08` controlled adapter test path.
- P0-IMPL-11 Asset Registry.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `implementation`
- `cpi`
- `controlled-adapter`
- `routes`

## Suggested Owner / 推奨Owner

Toby.

## Notes / 補足

Do not claim this work proves Orca/Raydium readiness. Production venue readiness is a separate mainnet launch gate.

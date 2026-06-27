# [P0-IMPL-24] Implement pause, asset disable, and route disable controls / 一時停止・資産無効化・ルート無効化コントロールを実装

## Summary / 概要

Implement P0 safety controls for market pause, asset disable, route disable, and exit-only emergency behavior.

The preferred emergency response is to block new risk while preserving safe user exits where route and pricing conditions allow.

## Requirement Areas / 関連Requirement Area

- ADMIN-001 to ADMIN-008
- POLICY-010, POLICY-011, POLICY-012
- MINT-002, MINT-007, MINT-008
- REDEEM-002, REDEEM-003, REDEEM-007, REDEEM-008
- PMV-008, PMV-013
- NFR-UPG-001, NFR-UPG-002

## Source Documents / 参照ドキュメント

- `requirements/03-mint-requirements.md`
- `requirements/04-redeem-requirements.md`
- `requirements/05-swap-cpi-execution-requirements.md`
- `requirements/07-execution-policy-risk-controls.md`
- `requirements/09-admin-safety-requirements.md`
- `requirements/12-pre-mainnet-validation-requirements.md`

## Scope / 対象範囲

- Implement market pause/unpause control.
- Enforce paused market blocks mint.
- Define and implement redeem behavior for paused/exit-only markets according to accepted policy.
- Implement asset flag updates by authorized role.
- Implement route disable/update by authorized route authority or the accepted P0 route config mechanism.
- Enforce independent asset flags:
  - creation_enabled
  - mint_enabled
  - redeem_enabled
  - rebalance_enabled
- Support emergency setting:
  - creation_enabled=false
  - mint_enabled=false
  - redeem_enabled=true when safe
  - rebalance_enabled=false
- Emit/log control changes where supported.

## Out of Scope / 対象外

- Full governance system.
- Multisig implementation unless already part of the accepted spec.
- Production monitoring.
- Auction/JIT disable controls.
- Frontend notification flows.

## Acceptance Criteria / 完了条件

- Unauthorized pause/unpause fails.
- Authorized pause/unpause succeeds.
- Paused market blocks mint.
- Exit-only asset config blocks new creation and mint while allowing redeem when safe.
- `redeem_enabled=false` blocks normal redeem.
- Route disable causes mint/redeem route validation to fail.
- Unauthorized asset update fails.
- Unauthorized route update fails.
- Emergency controls are covered by deterministic tests.

## Test Requirements / 必要なテスト

- Pause blocks mint test.
- Unpause restores mint test.
- Exit-only asset flag test.
- Redeem disabled failure test.
- Route disabled failure test.
- Unauthorized pause/update failure tests.
- Emergency control behavior test.

## Dependencies / 依存関係

- P0-IMPL-10 ProtocolConfig.
- P0-IMPL-11 Asset Registry.
- P0-IMPL-17 Controlled adapter interface or accepted route model.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `implementation`
- `admin`
- `safety`
- `risk-controls`

## Suggested Owner / 推奨Owner

Toby primary, ADP review.

## Notes / 補足

Emergency controls should avoid overblocking user exits. If redeem must be disabled, the reason should be explicit and test-covered.

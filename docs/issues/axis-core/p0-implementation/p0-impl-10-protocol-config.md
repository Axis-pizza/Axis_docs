# [P0-IMPL-10] Implement ProtocolConfig account and initialize instruction / ProtocolConfigアカウントと初期化命令を実装

## Summary / 概要

Implement the foundational `ProtocolConfig` account and protocol initialization instruction for Axis Core.

This account should define protocol authorities, the configured USDC mint, protocol treasury, and protocol-level fee configuration needed by later market, mint, redeem, asset, route, pricing, fee, and emergency-control instructions.

## Requirement Areas / 関連Requirement Area

- AXIS-CORE
- ADMIN-001
- ADMIN-008
- FEE-004, FEE-011, FEE-012
- NFR-SEC-005

## Source Documents / 参照ドキュメント

- `requirements/01-definitions-and-decision-log.md`
- `requirements/09-admin-safety-requirements.md`
- `requirements/13-fee-model-requirements.md`
- `requirements/19-axis-core-implementation-rfc.md`

## Scope / 対象範囲

- Define `ProtocolConfig` state.
- Add deterministic PDA derivation for protocol config.
- Implement initialize instruction and required authority checks.
- Store configured USDC mint.
- Store protocol authority fields:
  - authority
  - pause_authority
  - asset_registry_authority
  - route_registry_authority
  - pricing_registry_authority
  - protocol_treasury
- Store protocol-level fee config:
  - mint_fee_bps
  - redeem_fee_bps
  - creator_share_bps
  - protocol_share_bps
  - max_mint_fee_bps
  - max_redeem_fee_bps
- Enforce v1 defaults or validated configured values:
  - mint_fee_bps = 100 unless explicitly configured within caps
  - redeem_fee_bps = 0
  - creator_share_bps + protocol_share_bps = 10000
  - mint_fee_bps <= max_mint_fee_bps
  - redeem_fee_bps <= max_redeem_fee_bps

## Out of Scope / 対象外

- Asset registry implementation.
- DTF market creation.
- Fee claim instructions.
- Production deployment or upgrade-authority procedure.
- Backend or app configuration storage.

## Acceptance Criteria / 完了条件

- Protocol config can be initialized once at the expected PDA.
- Reinitialization fails.
- Unauthorized initialization or invalid authority inputs fail.
- Invalid fee share sum fails.
- Fee bps above protocol caps fail.
- `redeem_fee_bps` and `max_redeem_fee_bps` are zero for v1 unless a later approved requirement changes this.
- Protocol treasury and configured USDC mint are readable from account state.

## Test Requirements / 必要なテスト

- LiteSVM or unit test for successful initialize.
- Reinitialize failure test.
- Invalid fee config failure tests.
- Invalid authority or missing signer failure tests.
- State serialization/deserialization test if applicable to the chosen account model.

## Dependencies / 依存関係

- Existing `P0-CORE-00` scaffold.
- Existing `P0-SPEC-04` account model proposal.
- Existing `P0-SPEC-05` instruction surface proposal.
- Existing `P0-FEE-07` fee accounting proposal.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `implementation`
- `protocol-config`
- `admin`
- `fees`

## Suggested Owner / 推奨Owner

Toby primary, ADP review.

## Notes / 補足

This issue should not encode final mainnet deployment policy. It should implement the protocol state needed by the P0 contract lifecycle and keep open deployment traceability requirements in the mainnet launch gate track.

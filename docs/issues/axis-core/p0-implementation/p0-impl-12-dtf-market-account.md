# [P0-IMPL-12] Implement DTFMarket account and PDA derivation / DTFMarketアカウントとPDA導出を実装

## Summary / 概要

Implement the core `DTFMarket` account model and deterministic PDA derivation for Axis Core markets.

`DTFMarket` is the canonical on-chain market identifier. Backend strategy registries, app metadata, route plans, or legacy vault addresses must not replace it as protocol truth.

## Requirement Areas / 関連Requirement Area

- DTF-001 to DTF-017
- FEE-001 to FEE-005
- APPIF-001, APPIF-005
- RBA-026, RBA-035, RBA-036
- NFR-DOC-001

## Source Documents / 参照ドキュメント

- `requirements/01-definitions-and-decision-log.md`
- `requirements/02-dtf-market-requirements.md`
- `requirements/13-fee-model-requirements.md`
- `requirements/15-app-contract-interface-requirements.md`
- `requirements/16-route-builder-backend-api-requirements.md`
- `requirements/19-axis-core-implementation-rfc.md`

## Scope / 対象範囲

- Define `DTFMarket` state.
- Define deterministic PDA derivation and market seed strategy.
- Store creator and creator fee destination.
- Store DTF mint address.
- Store market asset list and target weights.
- Store reserve vault references or derivation inputs.
- Store market status at minimum:
  - Created
  - Active
  - Paused
  - Deprecated
- Store immutable market fee config copied or derived from ProtocolConfig at creation:
  - mint_fee_bps
  - redeem_fee_bps
  - creator_share_bps
  - protocol_share_bps
- Store accrued fee state or references if this belongs in the market account per final fee design.

## Out of Scope / 対象外

- Full `create_market` instruction validation.
- Actual reserve vault account creation.
- Mint/redeem instruction behavior.
- Auction activation state beyond future-compatible reserved fields if approved by account model review.
- Off-chain metadata schema.

## Acceptance Criteria / 完了条件

- Market PDA can be deterministically derived.
- Market account stores canonical DTF market identity and DTF mint.
- Market stores 2 to 5 assets and their target weights.
- Market stores immutable creator and creator fee destination.
- Market stores immutable market fee config derived from ProtocolConfig.
- Market status can be read by mint/redeem/pause flows.
- No market-level TVL cap is required.
- The account model does not encode legacy direct basket deposit/withdraw semantics.

## Test Requirements / 必要なテスト

- PDA derivation test.
- Market serialization/deserialization test.
- Market status read/update test if status updates are included in this issue.
- Immutable field test coverage may be completed in P0-IMPL-15 and P0-IMPL-24.

## Dependencies / 依存関係

- P0-IMPL-10 ProtocolConfig.
- P0-IMPL-11 Asset Registry.
- Existing `P0-SPEC-04` account model proposal.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `implementation`
- `dtf-market`
- `account-model`

## Suggested Owner / 推奨Owner

Toby primary, ADP review.

## Notes / 補足

Keep the account indexer-friendly, but do not make off-chain indexing part of protocol truth.

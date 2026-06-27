# [P0-IMPL-28] Add Devnet/Titan demo evidence script and README / Devnet・Titanデモ証跡スクリプトとREADMEを追加

## Summary / 概要

Add a demo evidence script and README that summarize contract-level proof for P0 Axis Core.

This evidence package should support Toby / ADP / Muse and partner conversations by showing create_market, mint, redeem, reserve vault balances, DTF Token-2022 mint/burn evidence, fee separation, controlled adapter execution, and known limitations.

## Requirement Areas / 関連Requirement Area

- PMV-014
- PMV-010
- APPIF-016
- BD / partner demo readiness
- NFR-OBS-001
- SECONDARY boundary context

## Source Documents / 参照ドキュメント

- `requirements/01-definitions-and-decision-log.md`
- `requirements/12-pre-mainnet-validation-requirements.md`
- `requirements/15-app-contract-interface-requirements.md`
- `requirements/17-auction-and-lvr-design-research.md`
- `requirements/18-secondary-market-and-clear-correction-requirements.md`
- `requirements/19-axis-core-implementation-rfc.md`

## Scope / 対象範囲

- Add a script or command that runs the deterministic P0 demo evidence path.
- Produce or document evidence for:
  - clean Axis Core repository state
  - create_market
  - mint
  - redeem
  - reserve vault balances before/after
  - DTF Token-2022 mint/burn
  - fee reserve separation
  - controlled adapter execution
  - known limitations
- Add README explaining how to run and interpret the evidence.
- Clearly label controlled adapter evidence as non-production venue readiness.
- Clearly label Titan as a distribution/routing partner narrative, not an Axis Core dependency.
- Clearly state Devnet demo does not equal mainnet readiness.

## Out of Scope / 対象外

- Titan integration inside Axis Core.
- Production Titan API, routing, or distribution implementation.
- Mainnet launch checklist completion.
- Production Orca/Raydium CPI readiness.
- Auction, ClearCorrection, or Axis-controlled JIT liquidity.
- Creating GitHub issues or PRs from the evidence script.

## Acceptance Criteria / 完了条件

- README can be shared with internal reviewers to understand the P0 demo.
- Script or documented command runs deterministic create_market / mint / redeem proof.
- Evidence includes reserve vault balances before/after.
- Evidence includes DTF Token-2022 mint/burn results.
- Evidence includes fee custody/accrual separation from reserves/NAV.
- Evidence includes controlled adapter execution proof.
- README documents known limitations and non-claims.
- README does not claim production venue readiness, mainnet readiness, or Titan integration.

## Test Requirements / 必要なテスト

- Evidence command runs locally.
- Evidence output is deterministic enough for reviewer comparison.
- README instructions are tested from a clean checkout or clean test fixture where practical.

## Dependencies / 依存関係

- P0-IMPL-25 Create_market + mint E2E test.
- P0-IMPL-26 Redeem E2E test.
- P0-IMPL-27 Negative tests.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `demo`
- `docs`
- `litesvm`
- `partner-evidence`

## Suggested Owner / 推奨Owner

Toby primary, Muse review.

## Notes / 補足

The demo proof is contract-level evidence. It should be useful for Titan conversations, but it must not define Titan integration as a P0 Axis Core requirement.

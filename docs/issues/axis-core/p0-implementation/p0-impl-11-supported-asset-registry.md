# [P0-IMPL-11] Implement supported asset registry and asset config state / サポート資産レジストリと資産設定状態を実装

## Summary / 概要

Implement Axis Core asset registry and per-asset execution policy state.

The registry should let Axis explicitly configure which assets may be used in DTF markets and which operations are enabled for each asset. Liquidity discovery or script output must not auto-enable assets.

## Requirement Areas / 関連Requirement Area

- ASSET
- POLICY-002 to POLICY-012
- DTF-006 to DTF-008
- MINT-005 to MINT-007
- REDEEM-007
- ADMIN-002, ADMIN-003
- NFR-UPG-002

## Source Documents / 参照ドキュメント

- `requirements/02-dtf-market-requirements.md`
- `requirements/03-mint-requirements.md`
- `requirements/04-redeem-requirements.md`
- `requirements/07-execution-policy-risk-controls.md`
- `requirements/08-asset-universe-requirements.md`
- `requirements/09-admin-safety-requirements.md`

## Scope / 対象範囲

- Define asset config/account state.
- Store asset mint, token program expectations, decimals if needed, support status, and policy flags:
  - creation_enabled
  - mint_enabled
  - redeem_enabled
  - rebalance_enabled
- Store execution limits:
  - hard_min_allocation_usdc
  - max_trade_usdc
  - max_weight_bps
  - max_price_impact_bps
  - max_pricing_deviation_bps
- Store pricing requirement and route requirement flags sufficient for P0 validation.
- Add authorized asset registration/update instructions or the minimal P0 instruction surface chosen by the spec.
- Enforce that new assets default to disabled unless explicitly configured otherwise by the authorized role.

## Out of Scope / 対象外

- Automated asset discovery scripts.
- Launch-ready asset promotion workflow automation.
- Full 500-asset universe.
- xStocks/StockToken special handling beyond flags needed to keep them disabled or manual-review-gated for P0.
- Production venue route validation.

## Acceptance Criteria / 完了条件

- Registered asset can be read and validated by create_market, mint, and redeem paths.
- Unknown asset fails market creation.
- `creation_enabled=false` blocks new DTF creation using that asset.
- `mint_enabled=false` blocks mint but does not automatically block redeem.
- `redeem_enabled=false` blocks normal redeem for that asset.
- Asset max weight is enforceable during market creation.
- Asset min/max trade policy is available to mint/redeem execution checks.
- Unauthorized asset config updates fail.

## Test Requirements / 必要なテスト

- Register asset success test.
- Unauthorized update failure test.
- Unknown asset failure test.
- Disabled creation/mint/redeem flag tests.
- Max weight failure test.
- Min allocation and max trade policy read tests.

## Dependencies / 依存関係

- P0-IMPL-10 ProtocolConfig.
- Existing account and instruction surface specs.

## Suggested Labels / 推奨ラベル

- `p0`
- `axis-core`
- `implementation`
- `asset-registry`
- `risk-controls`

## Suggested Owner / 推奨Owner

Toby primary, ADP review.

## Notes / 補足

The v0 supported candidate list in the asset requirements is a review input, not automatic launch approval. This issue should implement the on-chain enablement mechanism, not decide final launch assets.

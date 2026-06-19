# Axis v1 Open Design Docs

This directory is the source of truth for the Axis v1 Open Design.

Axis v1 is designed as an open DTF protocol on Solana. A DTF, or Dex Traded Fund, is a tradable position token backed by a basket of actual underlying reserve assets.

These docs intentionally separate product/protocol design, execution policy, program specs, and diagrams.

## Directory Structure

```txt
docs/
  README.md

  v1-open-design/
    00-overview.md
    01-dtf-model.md
    02-system-architecture.md
    03-mint-redeem-flow.md
    04-swap-cpi-execution.md
    05-pricing-nav-accounting.md
    06-execution-policy-risk-controls.md
    07-asset-universe.md
    08-fee-model.md
    09-rebalance-scope.md
    10-titan-integration-boundary.md
    11-open-questions.md

  diagrams/
    system-architecture.mmd
    dtf-lifecycle.mmd
    mint-flow.mmd
    redeem-flow.mmd
    swap-cpi-execution.mmd
    pricing-nav-flow.mmd
    state-er-diagram.mmd
    asset-policy-state-machine.mmd
    market-lifecycle-state-machine.mmd
    titan-integration-boundary.mmd

  specs/
    accounts.md
    instructions.md
    errors.md
    constants.md
    math.md
```

## Current Status

Finalized enough for implementation planning:

- DTF model
- mint / redeem accounting
- controlled CPI execution direction
- pricing source model
- execution policy / risk controls
- 500 asset universe philosophy

Still open:

- Fee model
- Rebalance scope
- exact pricing source implementation
- first production-approved CPI venue after Orca spike
- Titan integration details
- execution-readiness classification for the 500-asset universe

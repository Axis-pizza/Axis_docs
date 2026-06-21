# Axis v1 Open Design & Requirements

This repository is the source of truth for Axis v1 protocol design and implementation requirements.

Axis v1 is an open DTF protocol on Solana.

A DTF, or Dex Traded Fund, is a tradable position token backed by a basket of actual underlying reserve assets. Users mint DTFs with USDC, Axis composes the underlying reserve assets through controlled CPI execution, and users redeem DTFs back to USDC by unwinding the reserve assets.

## Start Here

Read these in order:

1. [Requirements Overview](./requirements/00-requirements-overview.md)
2. [Definitions & Decision Log](./requirements/01-definitions-and-decision-log.md)
3. [DTF Market Requirements](./requirements/02-dtf-market-requirements.md)
4. [Mint Requirements](./requirements/03-mint-requirements.md)
5. [Redeem Requirements](./requirements/04-redeem-requirements.md)
6. [Swap CPI Execution Requirements](./requirements/05-swap-cpi-execution-requirements.md)
7. [Pricing & NAV Requirements](./requirements/06-pricing-nav-requirements.md)
8. [Execution Policy & Risk Controls](./requirements/07-execution-policy-risk-controls.md)
9. [Asset Universe Requirements](./requirements/08-asset-universe-requirements.md)
10. [Admin / Safety Requirements](./requirements/09-admin-safety-requirements.md)
11. [Non-Functional Requirements](./requirements/10-non-functional-requirements.md)
12. [Traceability Matrix](./requirements/11-traceability-matrix.md)
13. [Pre-Mainnet Validation Requirements](./requirements/12-pre-mainnet-validation-requirements.md)
14. [Fee Model Requirements](./requirements/13-fee-model-requirements.md)
15. [Auction and LVR Design Research](./requirements/17-auction-and-lvr-design-research.md)
16. [Secondary Market and ClearCorrection Requirements](./requirements/18-secondary-market-and-clear-correction-requirements.md)

## Program Specs

- [Accounts Spec](./specs/accounts.md)
- [Instructions Spec](./specs/instructions.md)
- [Math Spec](./specs/math.md)
- [Error Spec](./specs/errors.md)
- [Constants Spec](./specs/constants.md)
- [Testing Spec](./specs/testing.md)

## Issue Blueprints

Use these as the starting point for GitHub issues:

- [Issue Breakdown](./issue-blueprints/00-issue-breakdown.md)
- [Protocol Config & Registry Issues](./issue-blueprints/01-protocol-config-and-registry-issues.md)
- [DTF Market Issues](./issue-blueprints/02-dtf-market-issues.md)
- [Mint / Redeem Issues](./issue-blueprints/03-mint-redeem-issues.md)
- [Swap CPI Adapter Issues](./issue-blueprints/04-swap-cpi-adapter-issues.md)
- [Pricing / NAV Issues](./issue-blueprints/05-pricing-nav-issues.md)
- [Testing / Security Issues](./issue-blueprints/06-testing-security-issues.md)

## Design Principle

```txt
Axis First. Titan Compatible. Controlled Execution.
```

Axis Core must be able to create, mint, redeem, and account for DTFs independently. External routers such as Titan may later route into Axis DTF markets, but they are not part of Axis Core.

## Current Status

The Axis v1 fee model is confirmed and required from launch (see [Fee Model Requirements](./requirements/13-fee-model-requirements.md)). Pre-mainnet validation does not require public Devnet (see [Pre-Mainnet Validation Requirements](./requirements/12-pre-mainnet-validation-requirements.md)).

Axis v1 also requires a launch-day secondary-market surface. Public third-party DTF/USDC pools are external liquidity and may be indexed or displayed, but they do not receive an Axis LVR-mitigation claim. Axis-controlled JIT liquidity and ClearCorrection are separately gated by technical-spike evidence (see [Secondary Market and ClearCorrection Requirements](./requirements/18-secondary-market-and-clear-correction-requirements.md)).

The following areas are intentionally not finalized yet:

- Rebalance scope
- Titan integration details
- Exact production CPI venue list after the Orca spike
- Launch-readiness classification of the full 500-asset universe
- Fee custody account layout and exact fee-claim instruction names (see fee model Open Questions)

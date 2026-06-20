# Issue Breakdown

This file groups implementation work into GitHub issues.

Each issue should include:

```txt
Requirement IDs
Design Docs
Implementation Scope
Acceptance Criteria
Out of Scope
Test Cases
Dependencies
```

## Milestone 1: Core State & Registries

```txt
1. Implement ProtocolConfig (including protocol fee config: mint/redeem fee bps, creator/protocol shares, fee caps)
2. Implement AssetRegistry
3. Implement AssetExecutionPolicy
4. Implement PricingSourceRegistry
5. Implement ApprovedRouteRegistry
```

## Milestone 2: DTF Market Creation

```txt
6. Implement DTFMarket account
7. Implement MarketAssetWeight / reserve account model
8. Implement CreateMarket instruction
9. Implement composition validation
10. Implement market status model
10b. Implement market fee state (creator, creator_fee_destination, fee snapshot, accrued fees, fee vault)
```

## Milestone 3: Pricing / NAV

```txt
11. Implement PricingSource validation
12. Implement NAV calculator
13. Implement initial NAV handling
14. Implement pricing deviation checks
```

## Milestone 4: CPI Spike

```txt
15. Implement generic venue adapter interface
16. Implement Orca Whirlpool CPI spike
17. Measure compute/account limits
18. Implement actual balance delta helper
```

## Milestone 5: Mint / Redeem

```txt
19. Implement Mint instruction
20. Implement mint fee deduction and allocation checks (from net_usdc_for_composition)
21. Implement mint accounting (actual added value, excludes fee)
21b. Implement creator/protocol fee accrual on mint
22. Implement Redeem instruction (no explicit redeem fee in v1)
23. Implement redeem share/pro-rata accounting
24. Implement min_out / min_usdc_out checks
24b. Implement fee claim instructions (creator + protocol)
```

## Milestone 6: Safety and Tests

```txt
25. Implement pause / unpause
26. Implement emergency asset flags
27. Implement test suite (including fee tests)
28. Implement integration tests
29. Implement route/policy/pricing negative tests
30. Implement pre-mainnet validation matrix (local / LiteSVM / fork / venue CPI), no mandatory public Devnet
31. Implement guarded mainnet launch controls
```

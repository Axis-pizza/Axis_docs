# Admin / Safety Requirements

## 1. Overview

Axis v1 Open Version still needs admin and emergency controls.

The goal is not to limit market TVL, but to stop unsafe actions quickly.

## 2. Requirements

### ADMIN-001: ProtocolConfig must define authorities

Recommended authorities:

```txt
authority
pause_authority
asset_registry_authority
route_registry_authority
pricing_registry_authority
protocol_treasury
```

### ADMIN-002: Asset policy must be updateable by authorized role

Acceptance criteria:

```txt
- unauthorized update fails
- authorized update succeeds
- update is logged/emitted
```

### ADMIN-003: Asset flags must be updateable quickly

Acceptance criteria:

```txt
- emergency can disable creation/mint
- redeem can remain enabled
```

### ADMIN-004: Approved routes must be updateable by route authority

Acceptance criteria:

```txt
- route can be added
- route can be disabled
- route cannot be changed by unauthorized user
```

### ADMIN-005: Pricing sources must be updateable by pricing authority

Acceptance criteria:

```txt
- pricing source can be added
- pricing source can be disabled
- stale/unsafe source can be replaced
```

### ADMIN-006: Market pause must be supported

Acceptance criteria:

```txt
- paused market blocks mint
- paused market may allow redeem depending on policy
- unpause restores normal behavior
```

### ADMIN-007: Dangerous asset response should prefer exit-only mode

Recommended flow:

```txt
1. Set creation_enabled=false
2. Set mint_enabled=false
3. Keep redeem_enabled=true if route/pricing are still valid
4. Disable rebalance
5. Later disable redeem only if redemption itself becomes unsafe
```

### ADMIN-008: Fee configuration administration must respect protocol rules

Protocol-level fee config is owned by the protocol authority and bounded by caps. Market-level fee config is derived from protocol config at creation and is immutable thereafter. Canonical definitions live in `13-fee-model-requirements.md` and `02-dtf-market-requirements.md`.

Acceptance criteria:

```txt
- admin/authority may manage protocol-level fee config (mint/redeem fee bps, creator/protocol shares) subject to caps
- mint_fee_bps <= max_mint_fee_bps and redeem_fee_bps <= max_redeem_fee_bps
- creator_share_bps + protocol_share_bps == 10000
- creators cannot customize fee bps per market
- market fee config is immutable after market creation
- fee custody is separate from reserve custody
- accrued fees are excluded from NAV and are not reserve backing
- unauthorized fee config change fails
```

## 3. Emergency Workflow

```mermaid
flowchart TD
    Detect[Detect asset / route / pricing risk] --> Decide[Decide emergency action]
    Decide --> Creation[Disable creation]
    Decide --> Mint[Disable mint]
    Decide --> Redeem{Is redeem safe?}
    Redeem -->|Yes| ExitOnly[Keep redeem enabled]
    Redeem -->|No| DisableRedeem[Disable redeem]
    ExitOnly --> Notify[Notify users / UI / indexer]
    DisableRedeem --> Notify
    Notify --> Review[Manual review and remediation]
```

## 4. Issue Candidates

```txt
- Implement ProtocolConfig account
- Implement authority validation
- Implement SetAssetExecutionPolicy
- Implement SetAssetExecutionFlags
- Implement RegisterApprovedRoute
- Implement DisableApprovedRoute
- Implement SetPricingSource
- Implement PauseMarket
- Implement UnpauseMarket
- Implement emergency tests
```

# Non-Functional Requirements

## 1. Security Requirements

### NFR-SEC-001: No unchecked external accounts

All accounts used for CPI execution, pricing, reserves, and mints must be validated.

### NFR-SEC-002: No blind trust in off-chain quotes

Off-chain quotes may guide execution, but final accounting must use:

```txt
- actual balance deltas
- approved pricing sources
- validated reserves
```

### NFR-SEC-003: CPI routes must be allowlisted

Arbitrary CPI to unknown programs is not allowed.

### NFR-SEC-004: Reserve accounts must be Axis-controlled

Reserve token accounts must be owned/controlled by the expected authority.

### NFR-SEC-005: Mint authority must be controlled

DTF mint authority must be controlled by Axis program authority model.

## 2. Compute Requirements

### NFR-CU-001: Compute must be measured per production venue

Compute and account usage must be measured for each production venue path — Orca Whirlpool and Raydium CPMM (see `05-swap-cpi-execution-requirements.md` EXEC-015..017):

```txt
- compute per swap (per venue)
- account count per swap (per venue)
- realistic max swaps per transaction
```

### NFR-CU-002: 5 asset DTF must be tested

Even if not all venues support 5 swaps in one transaction, the design must explicitly test feasibility.

### NFR-CU-003: Mainnet readiness requires production venue tests

Controlled adapter tests validate Axis accounting invariants but are not sufficient for mainnet readiness. At least two production venue CPI paths (Orca Whirlpool and Raydium CPMM fallback) must be tested before mainnet launch.

## 3. Reliability Requirements

### NFR-REL-001: Mint and redeem must be all-or-nothing

Partial success is not allowed.

### NFR-REL-002: Failed swap must revert full transaction

### NFR-REL-003: Failed min_out must revert full transaction

### NFR-REL-004: Tests must cover actual balance deltas and rollback behavior

Test coverage must verify that mint/redeem accounting uses actual token balance deltas (not quotes) and that failure paths roll back fully.

```txt
- actual reserve balance deltas verified on mint
- actual USDC balance delta verified on redeem
- failed mint accrues no fees and mints no DTF
- failed redeem does not permanently burn DTF or transfer reserves/USDC
```

## 4. Observability Requirements

### NFR-OBS-001: Key execution data should be logged

For mint/redeem:

```txt
market_id
user
gross input amount
mint_fee_usdc (mint)
creator_fee_usdc / protocol_fee_usdc (mint)
net_usdc_for_composition (mint)
asset allocations
actual received
NAV
minted/redeemed amount
```

Execution spread / slippage must be distinguishable from Axis fees in logs (see `13-fee-model-requirements.md` §13).

## 5. Upgrade and Maintenance Requirements

### NFR-UPG-001: Policy and route configuration should be updateable without redeploy

### NFR-UPG-002: Asset universe should be updateable through controlled authority

## 6. Documentation Requirements

### NFR-DOC-001: Every major protocol behavior must map to a requirement ID

### NFR-DOC-002: Every GitHub issue should reference requirement IDs

### NFR-DOC-003: Open questions must remain tracked until resolved

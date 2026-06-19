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

### NFR-CU-001: Compute must be measured per adapter

The Orca CPI spike must measure:

```txt
- compute per swap
- account count per swap
- realistic max swaps per transaction
```

### NFR-CU-002: 5 asset DTF must be tested

Even if not all venues support 5 swaps in one transaction, the design must explicitly test feasibility.

## 3. Reliability Requirements

### NFR-REL-001: Mint and redeem must be all-or-nothing

Partial success is not allowed.

### NFR-REL-002: Failed swap must revert full transaction

### NFR-REL-003: Failed min_out must revert full transaction

## 4. Observability Requirements

### NFR-OBS-001: Key execution data should be logged

For mint/redeem:

```txt
market_id
user
input amount
asset allocations
actual received
NAV
minted/redeemed amount
```

## 5. Upgrade and Maintenance Requirements

### NFR-UPG-001: Policy and route configuration should be updateable without redeploy

### NFR-UPG-002: Asset universe should be updateable through controlled authority

## 6. Documentation Requirements

### NFR-DOC-001: Every major protocol behavior must map to a requirement ID

### NFR-DOC-002: Every GitHub issue should reference requirement IDs

### NFR-DOC-003: Open questions must remain tracked until resolved

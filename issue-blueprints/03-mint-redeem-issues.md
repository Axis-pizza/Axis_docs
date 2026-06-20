# Mint / Redeem Issues

## Issue: Implement mint fee and allocation calculator

Requirement IDs:

```txt
MINT-003
MINT-004
MINT-005
FEE-006
FEE-007
```

Acceptance criteria:

```txt
- deducts mint fee from gross user USDC input before allocation
- computes net_usdc_for_composition = gross - mint fee
- computes allocation by weight from net_usdc_for_composition (not gross)
- rejects allocation below 1 USDC
- rejects allocation above max_trade_usdc
```

## Issue: Implement Mint instruction

Requirement IDs:

```txt
MINT-001
MINT-002
MINT-006
MINT-007
MINT-008
MINT-009
MINT-010
MINT-011
MINT-012
MINT-013
MINT-014
MINT-015
MINT-016
MINT-017
MINT-018
```

Acceptance criteria:

```txt
- USDC input only
- active market only
- mint_enabled assets only
- approved routes only
- min_out enforced
- actual balance delta used
- actual added value used (excludes mint fee)
- minted DTF based on net actual reserve value, not gross input or quote
- initial NAV handled
- accrues creator and protocol fees (claim-based)
- failed mint accrues no fees and mints no DTF
- fee custody kept separate from reserve custody
- all-or-nothing execution
```

## Issue: Implement redeem share calculator

Requirement IDs:

```txt
REDEEM-002
REDEEM-003
```

Acceptance criteria:

```txt
- calculates redeem share
- calculates pro-rata reserve amounts
```

## Issue: Implement Redeem instruction

Requirement IDs:

```txt
REDEEM-001
REDEEM-004
REDEEM-005
REDEEM-006
REDEEM-007
REDEEM-008
REDEEM-009
REDEEM-010
REDEEM-011
REDEEM-012
REDEEM-018
```

Acceptance criteria:

```txt
- validates DTF input mint
- redeem_enabled assets only
- approved routes only
- min_usdc_out enforced
- actual USDC delta used
- user_usdc_out = actual_usdc_received (no explicit redeem fee in v1)
- no creator or protocol fee accrues on redeem
- fee custody untouched by redeem
- burns/escrows DTF
- all-or-nothing execution
```

## Issue: Implement fee claim instructions

Requirement IDs:

```txt
FEE-013
FEE-014
FEE-015
```

Acceptance criteria:

```txt
- creator fee claim transfers accrued creator fee to creator_fee_destination
- protocol fee claim transfers accrued protocol fee to protocol_treasury
- only authorized claimant can claim
- claimed amount decremented from accrued balance
- double claim impossible
- claim does not change reserve balances

TODO: exact claim instruction names and whether creator/protocol claims are unified remain open (see 13-fee-model-requirements.md Open Questions).
```

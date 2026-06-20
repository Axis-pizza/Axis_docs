# Instructions Spec

## 1. InitializeProtocolConfig

Initializes global protocol configuration.

Inputs:

```txt
authority
pause_authority
asset_registry_authority
route_registry_authority
pricing_registry_authority
default_usdc_mint
protocol_treasury
mint_fee_bps              // = 100
redeem_fee_bps            // = 0
creator_share_bps         // = 4000
protocol_share_bps        // = 6000
max_mint_fee_bps          // = 300
max_redeem_fee_bps        // = 0
```

Acceptance criteria:

```txt
- config initialized once
- USDC mint stored
- authorities stored
- protocol fee config stored (mint/redeem fee bps, creator/protocol shares, fee caps)
- mint_fee_bps <= max_mint_fee_bps
- redeem_fee_bps <= max_redeem_fee_bps
- creator_share_bps + protocol_share_bps == 10000
```

## 2. RegisterAsset

Registers an asset in the AssetRegistry.

Inputs:

```txt
asset_mint
category
decimals
manual_review_required
is_universe_asset
```

Checks:

```txt
authorized asset_registry_authority
asset mint not already registered
```

## 3. SetAssetExecutionPolicy

Creates or updates policy for an asset.

Inputs:

```txt
asset_mint
flags
hard_min_allocation_usdc
max_trade_usdc
max_weight_bps
max_price_impact_bps
pricing_requirement
max_pricing_deviation_bps
approved_route_required
manual_review_required
```

Checks:

```txt
authorized asset_registry_authority
asset registered
```

## 4. RegisterApprovedRoute

Registers an approved CPI route.

Inputs:

```txt
asset_mint
quote_mint
direction
venue_type
venue_program_id
pool_id
input_mint
output_mint
limits
```

Checks:

```txt
authorized route_registry_authority
asset registered
venue program allowed
input/output mint direction valid
```

## 5. SetPricingSource

Creates or updates pricing source.

Inputs:

```txt
asset_mint
source_type
source_account
quote_mint
max_staleness_slots
max_deviation_bps
enabled
```

Checks:

```txt
authorized pricing_registry_authority
asset registered
source type compatible with asset policy
```

## 6. CreateMarket

Creates a DTF market.

Inputs:

```txt
creator
asset_mints[]
weights_bps[]
creator_fee_destination
optional_initial_seed_usdc
```

Note: market fee bps are NOT a creator input. Market fee config is derived from protocol-level fee config (or a protocol-approved preset) at creation time and is immutable afterward. The creator only supplies `creator_fee_destination`.

Checks:

```txt
asset_count >= 2
asset_count <= 5
no duplicate assets
sum(weights_bps) == 10000
each weight >= 100
each asset registered
each asset creation_enabled
each weight <= asset.max_weight_bps
pricing source exists
route readiness requirement satisfied
```

Outputs:

```txt
DTFMarket
DTF mint
reserve token accounts
market asset weights
market fee state (snapshot of protocol fee config + creator, creator_fee_destination)
USDC fee vault (custody separate from reserve token accounts)
```

## 7. Mint

Mints DTF with USDC.

Inputs:

```txt
market
usdc_amount_in
route_plan[]
min_outs[]
user_source_usdc
user_destination_dtf
```

Checks:

```txt
market active
user input mint == USDC
each asset mint_enabled
allocation >= 1 USDC
allocation <= max_trade_usdc
route approved
price impact within max
pricing source valid
actual_received >= min_out
```

Behavior:

```txt
take gross_user_usdc_in
deduct mint_fee_usdc (retain in fee custody, not reserve custody)
split mint fee into creator_fee_usdc and protocol_fee_usdc
compute net_usdc_for_composition = gross_user_usdc_in - mint_fee_usdc
execute CPI swaps using net_usdc_for_composition only
measure actual reserve deltas
compute actual_added_value (excludes mint fee)
compute minted_dtf = actual_added_value / pre_trade_nav
mint DTF to user
accrue creator_fee_usdc and protocol_fee_usdc (claim-based, not transferred now)
```

Notes:

```txt
- minted_dtf is based on actual added reserve value, never on gross input or quote
- failed mint accrues no fees and mints no DTF
- accrued fees are excluded from NAV and reserve backing
```

## 8. Redeem

Redeems DTF back to USDC.

Inputs:

```txt
market
dtf_amount_in
route_plan[]
min_usdc_out
user_source_dtf
user_destination_usdc
```

Checks:

```txt
DTF mint matches market
redeem allowed
each asset redeem_enabled
route approved
actual_usdc_received >= min_usdc_out
```

Behavior:

```txt
compute redeem share
compute pro-rata asset amounts
execute CPI swaps back to USDC
measure actual USDC delta
burn DTF
transfer USDC out
```

## 9. PauseMarket

Pauses a market.

Checks:

```txt
authorized pause_authority
```

## 10. UnpauseMarket

Unpauses a market.

Checks:

```txt
authorized pause_authority
```

## 11. Rebalance

Status: not finalized.

Potentially v1.1.

## 12. Fee Claims

Axis v1 fees are claim-based (accrued during mint, transferred only on explicit claim). Two explicit claim paths are required. See `requirements/13-fee-model-requirements.md` (FEE-014, FEE-015).

ClaimCreatorFee:

```txt
- only the authorized creator_fee_destination / defined authority can claim
- transfers accrued_creator_fee_usdc to creator_fee_destination
- subtracts claimed amount from accrued_creator_fee_usdc
- double claim impossible
- does not change reserve balances
```

ClaimProtocolFee:

```txt
- only protocol_treasury authority can claim or sweep
- transfers accrued_protocol_fee_usdc to protocol_treasury
- subtracts claimed amount from accrued_protocol_fee_usdc
- double claim impossible
- does not change reserve balances
```

TODO / Open Question (per doc 13): exact claim instruction names, and whether creator and protocol claims are separate or unified, are not yet finalized. Names above are placeholders matching the requirements doc.

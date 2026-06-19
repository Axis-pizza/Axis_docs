# Fee Model

## 1. Status

Fee model is not finalized.

This document records the current design direction and open questions.

## 2. Fee Surfaces

Axis v1 may include:

```txt
market_creation_fee
mint_fee_bps
redeem_fee_bps
rebalance_fee_bps
protocol_share_bps
creator_share_bps
```

## 3. Current Direction

Initial market creation should likely have no fee.

Reason:

```txt
Creation is supply-side market formation.
Charging too early may reduce creator activity.
```

Normal mint and redeem may charge fees.

Potential direction:

```txt
initial market creation: no fee
normal mint: fee enabled
redeem: fee enabled
creator revenue share: enabled
protocol revenue share: enabled
```

## 4. Creator Incentive

Axis should preserve creator incentives.

One possible model:

```txt
creator receives a share of mint/redeem fees
protocol receives the remaining share
```

This encourages creators to share DTFs and drive distribution.

## 5. Open Questions

```txt
1. mint_fee_bps
2. redeem_fee_bps
3. creator_share_bps
4. protocol_share_bps
5. fee destination
6. fee accounting
7. whether fee should be taken before or after swap execution
8. whether creator share applies to both mint and redeem
```

# Rebalance Scope

## 1. Status

Rebalance scope is not finalized.

The current design should avoid overloading v1 with full automated rebalance complexity.

## 2. Target vs Actual Weights

Each DTF has target weights.

Actual reserve weights may drift due to:

- price movement
- mint execution
- redeem execution
- price impact
- fees
- routing differences

## 3. Current Direction

v1 should track:

```txt
target weights
actual reserve balances
actual weights
drift
```

However, fully automated rebalancing may be out of scope for v1.

## 4. Options

### Option A: Display Drift Only

```txt
v1 displays drift but does not rebalance
```

### Option B: Manual Rebalance

```txt
creator/admin/keeper can call rebalance instruction
```

### Option C: Mint/Redeem-Aware Soft Rebalance

```txt
mint/redeem execution partially offsets drift
```

### Option D: Full Automated Rebalance

```txt
keeper or protocol automatically rebalances DTFs
```

## 5. Recommended v1 Direction

```txt
- track target weights
- track actual weights
- display drift
- support manual or limited rebalance later
- do not require full automatic rebalance for initial launch
```

## 6. Open Questions

```txt
1. Is rebalance included in v1 or v1.1?
2. Who can trigger rebalance?
3. Is rebalance permissionless or creator-only?
4. Are rebalance fees charged?
5. How is rebalance route safety verified?
6. How does rebalance interact with asset disable flags?
```

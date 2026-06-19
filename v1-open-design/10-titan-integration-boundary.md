# Titan Integration Boundary

## 1. Status

Titan is not Axis Core.

Axis v1 must be able to create, mint, redeem, and account for DTFs independently.

Titan may later integrate Axis DTF markets as an external routing / distribution layer.

## 2. Core Boundary

```txt
Axis Core:
  - DTF issuance
  - reserve accounting
  - mint/redeem
  - execution policy
  - pricing checks
  - approved CPI execution

Titan:
  - external routing
  - distribution
  - route discovery
  - integration surface for DTF markets
```

## 3. Integration Direction

Titan can potentially treat Axis DTF mint/redeem as a venue.

Example:

```txt
USDC -> Axis DTF
Axis DTF -> USDC
```

Titan may route users into Axis markets, similar to how it routes into other venues.

## 4. What Titan Should Not Be

Titan should not be required for:

- creating DTFs
- minting DTFs
- redeeming DTFs
- maintaining Axis reserve accounting
- enforcing Axis risk policy

## 5. Why This Matters

If Axis depends on Titan as core execution, Axis becomes a frontend or route wrapper.

Axis should remain a standalone DTF protocol.

## 6. Future Integration Questions

```txt
1. Should Titan see each DTF as USDC <-> DTF venue?
2. How should Titan quote Axis mint/redeem?
3. How should Titan-generated instructions map to Axis execution?
4. Does Titan need exact quote/execution matching?
5. Is Titan integration required for v1 launch or v1.1?
```

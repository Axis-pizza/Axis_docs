# Error Spec

## Market Errors

```txt
InvalidAssetCount
DuplicateAsset
InvalidWeightSum
WeightBelowGlobalMinimum
WeightAboveAssetMaximum
MarketPaused
MarketNotActive
OneAssetDtfNotAllowed
```

## Asset Policy Errors

```txt
AssetNotInUniverse
AssetCreationDisabled
AssetMintDisabled
AssetRedeemDisabled
AssetRebalanceDisabled
AllocationBelowMinimum
TradeAboveAssetMaximum
PriceImpactExceeded
ManualReviewRequired
```

## Route Errors

```txt
ApprovedRouteRequired
RouteNotApproved
VenueNotApproved
PoolNotApproved
InputMintMismatch
OutputMintMismatch
InvalidRouteDirection
TooManyRouteAccounts
```

## Pricing Errors

```txt
PricingSourceRequired
PricingSourceDisabled
StalePrice
OracleConfidenceTooWide
PricingDeviationExceeded
InvalidPrice
MissingTwap
MissingOracle
```

## Execution Errors

```txt
SwapCpiFailed
ActualReceivedBelowMinOut
InvalidBalanceDelta
InsufficientReserveBalance
InsufficientUsdcOut
```

## Accounting Errors

```txt
InvalidNav
ZeroSupplyUnexpected
InvalidRedeemShare
MathOverflow
RoundingError
```

## Authority Errors

```txt
InvalidAuthority
UnauthorizedPolicyUpdate
UnauthorizedRouteUpdate
UnauthorizedPricingUpdate
UnauthorizedPause
```

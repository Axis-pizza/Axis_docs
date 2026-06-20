# Error Spec

## Market Errors

```txt
InvalidAssetCount
OneAssetDtfNotAllowed
TooManyAssets
DuplicateAsset
InvalidWeightSum
WeightBelowGlobalMinimum
WeightAboveAssetMaximum
MarketPaused
MarketDeprecated
MarketNotActive
```

## Asset Errors

```txt
AssetNotRegistered
AssetNotInUniverse
AssetCreationDisabled
AssetMintDisabled
AssetRedeemDisabled
AssetRebalanceDisabled
ManualReviewRequired
```

## Policy Errors

```txt
AllocationBelowMinimum
TradeAboveAssetMaximum
PriceImpactExceeded
PricingDeviationExceeded
```

## Route Errors

```txt
ApprovedRouteRequired
RouteNotApproved
RouteDisabled
VenueNotApproved
PoolNotApproved
InputMintMismatch
OutputMintMismatch
InvalidRouteDirection
TooManyRouteAccounts
UnsupportedVenue
```

## Pricing Errors

```txt
PricingSourceRequired
PricingSourceDisabled
StalePrice
OracleConfidenceTooWide
InvalidPrice
MissingTwap
MissingOracle
StableDepegDetected
InvalidLstExchangeRate
StockTokenPricingNotAllowed
```

## Execution Errors

```txt
SwapCpiFailed
ActualReceivedBelowMinOut
InvalidBalanceDelta
InsufficientReserveBalance
InsufficientUsdcOut
UnexpectedTokenAccountMint
UnexpectedTokenAccountOwner
```

## Accounting Errors

```txt
InvalidNav
ZeroSupplyUnexpected
InvalidRedeemShare
MathOverflow
RoundingError
```

## Fee Errors

```txt
InvalidFeeConfig
FeeShareSumInvalid
MintFeeCapExceeded
RedeemFeeNotAllowed
CreatorFeeCustomizationNotAllowed
MarketFeeConfigImmutable
InvalidCreatorFeeDestination
InvalidProtocolTreasury
DoubleFeeClaim
FeeCustodyReserveOverlap
```

## Authority Errors

```txt
InvalidAuthority
UnauthorizedPolicyUpdate
UnauthorizedRouteUpdate
UnauthorizedPricingUpdate
UnauthorizedPause
UnauthorizedCreatorFeeClaim
UnauthorizedProtocolFeeClaim
```

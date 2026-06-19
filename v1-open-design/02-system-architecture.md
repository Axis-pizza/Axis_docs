# System Architecture

## 1. High-Level Architecture

Axis v1 consists of the following conceptual components:

```txt
Axis Core Program
  - DTF market lifecycle
  - DTF mint / redeem
  - reserve accounting
  - execution policy verification
  - pricing verification
  - CPI swap dispatch

Asset Universe / Registry
  - curated 500 asset universe
  - asset metadata
  - asset execution policy
  - pricing source requirements

Approved Route Registry
  - approved venue
  - approved pool
  - input / output mint pair
  - route-level limits

Pricing Source Registry
  - oracle
  - DEX TWAP
  - approved DEX spot
  - stable peg
  - LST exchange rate
  - StockToken-specific source

Approved Venue Adapters
  - Orca Whirlpool
  - Raydium CPMM
  - PumpSwap
  - Raydium CLMM
  - Meteora DLMM
```

## 2. Axis Core

Axis Core is responsible for deterministic accounting and execution validation.

It should not behave as a general-purpose DEX aggregator. It should be a controlled execution engine specialized for DTF mint/redeem.

```txt
Route discovery / quote / account assembly = off-chain
Execution / verification / accounting       = on-chain
```

## 3. Adapter-Based Venue Design

```txt
Axis Core
  -> Approved Venue Adapter
    -> Orca Whirlpool
    -> Raydium CPMM
    -> PumpSwap
    -> Raydium CLMM
    -> Meteora DLMM
```

Orca Whirlpool is the first CPI spike candidate because the team has direct access to Orca for technical guidance.

However, Axis Core must not depend only on Orca. The architecture remains venue-agnostic and adapter-based.

## 4. Titan Boundary

Titan is not Axis Core.

Titan may later integrate Axis DTF markets as a routeable venue, but Axis must be able to create, mint, redeem, and account for DTFs independently.

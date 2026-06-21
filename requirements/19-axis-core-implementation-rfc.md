# 19. Axis Core Implementation RFC

## 1. Purpose

This document is an RFC for defining the implementation strategy for Axis v1 Core.

The existing requirements documents define what Axis v1 should achieve. This document defines how we intend to implement those requirements in practice.

This is not a finalized technical specification.

This RFC intentionally separates:

* **Decision**: Items that can be treated as agreed implementation direction
* **Proposal**: Recommended direction that still requires engineering review
* **Validation Required**: Items that must be validated through implementation, testing, or mainnet-fork experiments
* **Open Question**: Items that should remain unresolved until further review

The purpose of this RFC is not to lock down exact account layouts, CPI implementations, or mathematical formulas immediately.
The purpose is to define the implementation boundary, product invariants, P0/P1/P2 scope, and unresolved technical questions so that Axis Core can be converted into actionable development tasks.

---

## 2. Background

Axis currently has existing programs such as `axis-vault` and `pfda-amm-3`.

However, the current Axis v1 requirements redefine Axis as a reserve-backed DTF protocol where USDC is the entry and exit asset.

The new Axis v1 Core aims to support the following flow:

1. A user mints a DTF with USDC
2. Axis Core composes the underlying assets through Solana venues
3. The underlying assets are held in program-controlled reserve vaults
4. The user receives a reserve-backed DTF token
5. On redeem, the user burns the DTF token, the reserve assets are unwound back into USDC, and the user receives USDC

The existing `axis-vault` program is closer to a legacy ETF wrapper where users directly deposit and withdraw multiple basket assets.

Because of this, the new USDC-in / USDC-out Axis Core should not be implemented as an incremental mutation of the existing `axis-vault` program. Instead, it should be implemented as a clean new repository and program.

---

## 3. Implementation Strategy Decisions

### Decision 1: New Repository

Axis v1 Core will be implemented in a new repository named `axis-core`.

The existing contract repository will not be treated as the main production implementation target for Axis v1.

Rationale:

* Existing `axis-vault` follows a direct basket deposit / withdraw model
* New Axis v1 follows a USDC mint / redeem reserve-backed DTF model
* Extending the existing implementation risks mixing legacy logic with the new design
* The account model, instruction model, accounting model, fee model, and route validation model are materially different
* A new repository is cleaner for implementation, review, audits, and external communication

---

### Decision 2: Legacy Repository Treatment

Existing implementations such as `axis-vault`, `pfda-amm-3`, and other legacy code should be treated as reference-only.

They should not become production dependencies by default.

Reference usage:

* `axis-vault`: reference for PDA-owned vault patterns, SPL Token CPI, pause/admin patterns, and selected test structure
* `pfda-amm-3`: reference for auction, PFDA, oracle freshness, and batch clearing research
* legacy frontend / IDL / Jupiter flows: not production interfaces for the new Axis v1 Core

Unless explicitly reviewed and approved, the new Axis Core should not depend on existing programs.

---

### Decision 3: Contract-first Scope

The first phase is contract-only.

The following areas are deferred for the current phase:

* backend architecture
* frontend redesign
* database schema
* ER diagram
* AWS infrastructure
* indexer architecture
* production Auction Program
* ClearCorrection
* Axis-controlled JIT liquidity

However, on-chain accounts should still be designed to be indexer-friendly so that future app, backend, and indexer integrations can read them reliably.

---

### Decision 4: Repository Layout

The initial repository layout will be:

```txt
axis-core/
  programs/
    axis-core/
  crates/
    axis-core-client/
    axis-core-test-utils/
  tests/
  sdk/
  docs/
```

Responsibilities:

```txt
programs/axis-core/
  Production on-chain Axis Core program

crates/axis-core-client/
  Rust client, instruction builders, account parsers

crates/axis-core-test-utils/
  LiteSVM helpers, fixtures, mock token setup, test account builders

tests/
  Integration tests and scenario tests

sdk/
  TypeScript SDK or generated client for future app/backend integration

docs/
  Requirements copy, implementation RFCs, account model, instruction model, testing plans
```

Backend, app, and database directories are intentionally not included in the first phase.

---

## 4. Technical Stack

### Decision

Axis Core will use Rust + Pinocchio / no_std as the primary implementation stack.

Expected stack:

```txt
On-chain:
- Rust
- Pinocchio / no_std
- SPL Token
- Token-2022
- PDA-owned reserve accounts

Testing:
- LiteSVM
- local validator
- Surfpool mainnet-fork
- later small-scale mainnet smoke tests

Client / SDK:
- Rust client crate
- TypeScript SDK
```

### Rationale

Reasons to use Pinocchio / no_std:

* It matches the existing team’s implementation context
* It gives fine-grained control over compute and account validation
* It is well-suited for reserve accounting, CPI validation, and ApprovedRoute validation
* It presents Axis as a low-level Solana-native program

Exact module structure and helper crate design remain subject to engineering review.

---

## 5. Product Invariants

This section defines product-level invariants that Axis v1 Core must preserve regardless of the final implementation details.

### Invariant 1: USDC Mint Entry

Users mint DTFs with USDC.

Axis v1 Core is not designed around users directly depositing multiple underlying basket assets.
The primary entry asset is USDC.

---

### Invariant 2: USDC Redeem Exit

Users burn DTF tokens and receive USDC as the final output.

Redeem should not primarily return each underlying basket asset directly to the user.
The core redeem flow unwinds reserve assets back into USDC.

---

### Invariant 3: Reserve-backed DTF Token

A DTF token is not merely a display token or a synthetic index token.

A DTF token is backed by real assets held in program-controlled reserve vaults.

---

### Invariant 4: Program-controlled Reserve Vaults

The underlying assets of a DTF are held in on-chain vaults controlled by Axis Core.

They are not held in the creator’s wallet, backend database, or frontend state.

Reserve custody is protocol truth and cannot be replaced by off-chain metadata or frontend display.

---

### Invariant 5: Mint Amount Must Not Rely Only on Estimates

The DTF mint amount must not be determined solely by frontend or backend estimates.

Axis Core must account for the value actually added to reserves, or the actual balance delta observed during execution.

The exact mint calculation formula requires engineering review.

---

### Invariant 6: Redeem Output Must Be Based on Actual USDC Received

The user’s redeem output should be based on the actual amount of USDC received, not only on a quoted or expected amount.

A slippage protection mechanism such as `min_usdc_out` should be considered mandatory.

The exact redeem accounting formula requires engineering review.

---

### Invariant 7: Fees Must Not Corrupt Reserve / NAV Accounting

Creator fees and protocol fees must not be mixed with reserve assets or NAV accounting.

Reserve assets are backing assets for DTF holders.
Fees are revenue for creators and the protocol.

These accounting domains must remain separate.

Baseline v1 fee model direction:

* mint-side fee is the default direction
* redeem fee should be zero by default
* creator / protocol split should exist
* fees should use claim-based accrual
* fees must not be included in reserve value or NAV

The exact fee account structure requires engineering review.

---

### Invariant 8: Backend / Route Builder Cannot Bypass On-chain Validation

Even if a backend, route builder, or external execution partner prepares a route plan, Axis Core must validate safety on-chain.

Axis Core must validate at least:

* the route is approved
* venue / pool / token accounts match expectations
* min_out constraints are satisfied
* reserve balance deltas are correct
* fee and accounting invariants are preserved

The backend is not protocol truth.

---

### Invariant 9: Public Secondary Pools Are External Liquidity

A DTF/USDC pool created on Orca, Raydium, or another external venue should be treated as external liquidity.

A public secondary pool may be useful for trading, but it does not imply Axis-native LVR protection.

Only auction / JIT liquidity flows explicitly activated by Axis may be described as Axis-native LVR mitigation.

---

### Invariant 10: Auction / ClearCorrection / JIT Are Not P0 Core Requirements

Auction Program, ClearCorrection, and Axis-controlled JIT liquidity are not P0 launch blockers for the initial Axis Core.

They remain spike / v1.1 / research scope.

The top priority for Axis v1 is to implement a safe reserve-backed DTF Core.

---

### Invariant 11: Axis Is Not Investment Advice

Axis is not an investment advisor and does not guarantee profit.

Axis is infrastructure for creating and trading theme-based or basket-based DTF exposure.

User outcomes depend on underlying assets, market price, liquidity, slippage, and external market conditions.

---

### Invariant 12: DB / Frontend Are Not Protocol Truth

Backend, database, and frontend systems are used for display, indexing, UX, and analytics.

The following protocol truth should be derived from on-chain state or verified transaction results:

* DTF market address
* DTF mint
* reserve vaults
* reserve balances
* DTF supply
* fee ownership / fee accrual
* mint/redeem accounting
* ApprovedRoute truth
* Auction winner truth, if later enabled

---

## 6. Token Standard Decision

### Proposal

DTF tokens should use Token-2022 mints as the default candidate.

### Rationale

Reasons to consider Token-2022 for DTF tokens:

* Better future compatibility with metadata and extensions
* More expressive asset design for DTF tokens
* Potential compatibility with tokenized stocks, RWAs, and future asset classes
* A new protocol primitive does not need to be constrained to legacy SPL Token mints by default

### v1 Conservative Policy

v1 should use Token-2022 conservatively.

Potentially acceptable for v1:

* Token-2022 mint
* metadata-related extension
* metadata pointer / token metadata, if safe

Avoid by default in v1:

* transfer fee
* transfer hook
* permanent delegate
* confidential transfer
* interest-bearing behavior
* default frozen account behavior

### Validation Required

Token-2022 adoption requires validation for:

* basic Token-2022 DTF mint / burn behavior
* wallet / indexer compatibility
* Orca / Raydium compatibility
* secondary market usability
* constraints when not using transfer-related extensions
* Surfpool mainnet-fork venue compatibility

---

## 7. Mainnet-fork Testing Strategy

### Decision

Surfpool should be used for Axis Core mainnet-fork validation.

### Scope

Surfpool should be used to validate:

* Token-2022 DTF mint behavior
* Orca Whirlpool CPI feasibility
* Raydium CPMM compatibility
* account layout / account count limits
* compute usage
* transaction size
* venue account validation
* realistic token mint / pool / vault behavior

### Testing Layers

```txt
LiteSVM:
  - deterministic unit/integration tests
  - account layout tests
  - fee math tests
  - mint/redeem accounting tests
  - failure path tests

Local validator:
  - basic SPL / Token-2022 lifecycle
  - program deployment flow
  - local integration flows

Surfpool:
  - mainnet-fork venue testing
  - Orca/Raydium CPI
  - Token-2022 compatibility
  - realistic account constraints

Small mainnet smoke:
  - final gated validation with strict caps
```

Public Devnet is not a required validation path.

---

## 8. P0 / P1 / P2 Scope

### P0: Core Launch Blockers

P0 includes what must exist for Axis v1 Core to function as a minimal reserve-backed DTF protocol.

```txt
P0:
- new `axis-core` repository
- Rust workspace / program scaffold
- Pinocchio / no_std Axis Core program
- Token-2022 DTF mint candidate
- ProtocolConfig concept
- DTFMarket concept
- reserve vault custody
- USDC mint entry
- USDC redeem exit
- DTF mint / burn
- creator / protocol mint fee model
- fee accounting separated from reserve/NAV
- asset registry / asset allowlist concept
- pricing source concept
- approved route concept
- at least one validated execution path
- actual balance delta verification
- min_out protection
- LiteSVM tests
- Surfpool validation plan
```

The purpose of P0 is not to complete every production venue, auction mechanism, or app experience.

The purpose of P0 is to prove that Axis Core can function as a reserve-backed DTF mint/redeem protocol.

---

### P1: Launch-preferred / Near-term Follow-up

```txt
P1:
- Raydium fallback path
- richer route validation
- TypeScript SDK polish
- app/backend integration docs
- metadata/indexer design
- secondary market display surface
- external pool indexing plan
- improved analytics snapshot plan
```

P1 items improve launch quality and distribution, but are separated from the minimal P0 Core.

---

### P2: Spike / v1.1 / Research

```txt
P2:
- Axis Auction Program
- ClearCorrection
- Axis-controlled JIT liquidity
- Orca DTF/USDC secondary settlement spike
- Jito bundle fallback research
- AuctionRevenueVault
- DFlow / Titan deeper execution partnership
- advanced secondary-market design
```

P2 items are important future extensions, but they should not block the first Axis Core implementation.

---

## 9. Contract Responsibilities

### Axis Core Is Responsible For

Axis Core is responsible for:

```txt
- protocol initialization
- DTF market creation
- DTF Token-2022 mint creation or validation
- reserve vault creation and validation
- USDC intake
- underlying asset composition
- DTF minting
- DTF burning
- reserve unwind
- USDC payout
- fee accrual
- creator/protocol fee split
- route validation
- pricing/NAV validation framework
- pause / admin controls
- balance delta validation
```

### Axis Core Is Not Responsible For

Axis Core is not responsible for:

```txt
- investment advice
- profit guarantees
- frontend portfolio display
- off-chain metadata truth
- DB analytics truth
- public pool price guarantees
- auction winner selection in v1
- ClearCorrection in P0
- external pool LVR protection
```

---

## 10. Deferred Areas

The following areas are not designed in detail in the current phase:

```txt
- backend architecture
- database schema
- ER diagram
- AWS deployment architecture
- indexer architecture
- frontend redesign
- production Auction Program
- ClearCorrection
- JIT liquidity
- Jito bundle integration
```

However, contract accounts should still be designed so that future app, backend, and indexer systems can read them easily.

---

## 11. Engineering Review Required

The following items should not be finalized by founder/product judgment alone.

They require engineering review or spike validation.

```txt
- exact account layout
- exact instruction account ordering
- Token-2022 extension set
- mint amount calculation
- redeem accounting formula
- NAV calculation method
- pricing source registry design
- ApprovedRoute validation model
- decimals normalization
- rounding / dust handling
- fee accrual account structure
- Orca Whirlpool CPI implementation
- Raydium CPMM compatibility
- compute / account / transaction size limits
- Surfpool-based test setup
```

---

## 12. Open Questions

Current open questions:

```txt
1. Which metadata extension should be used for the DTF Token-2022 mint?
2. How much of PricingSourceRegistry should be implemented in P0?
3. Should ApprovedRoute accounts be route-based, asset-pair-based, venue-based, or pool-based?
4. Should Orca Whirlpool be a P0 production path, or a P0 spike + P1 production path?
5. Should Raydium CPMM fallback be P0 or P1?
6. How should initial NAV / initial supply be determined on first mint?
7. How should mint/redeem rounding and dust be handled?
8. Should fee accrual be stored at the market level, or split into separate creator/protocol claim states?
9. Where should the boundary between on-chain metadata and off-chain metadata be?
10. Should public secondary pool indexing be included in contract scope, or fully separated into backend/indexer scope?
```

---

## 13. Development Task Readiness

This RFC is ready to be converted into development tasks once the following are true:

```txt
- Implementation strategy is agreed
- Product invariants are agreed
- P0 / P1 / P2 scope is agreed
- Open questions are separated from decisions
- Engineering review items are explicit
```

Not all account layouts or CPI details need to be finalized before development begins.

The following tasks can start early:

```txt
- new repository scaffold
- Rust workspace setup
- program scaffold
- client crate scaffold
- test-utils scaffold
- LiteSVM test scaffold
- CI setup
- Token-2022 basic mint/burn test
- Surfpool feasibility setup
- account model proposal
- instruction surface proposal
```

---

## 14. Proposed Initial Milestones

### Milestone 0: Repository Scaffold

```txt
- Create `axis-core` repository
- Set up Rust workspace
- Add `programs/axis-core`
- Add `crates/axis-core-client`
- Add `crates/axis-core-test-utils`
- Add `tests`
- Add `sdk`
- Add `docs`
- Add CI for fmt/build/test
```

### Milestone 1: Core Account / Instruction Proposal

```txt
- Draft ProtocolConfig
- Draft DTFMarket
- Draft AssetConfig
- Draft PricingSource
- Draft ApprovedRoute
- Draft fee accounting state
- Draft instruction surface
```

### Milestone 2: Token / Vault Foundation

```txt
- Token-2022 DTF mint test
- reserve vault creation test
- USDC vault / intake test
- basic mint/burn skeleton
```

### Milestone 3: Mint/Redeem Accounting

```txt
- mint with USDC skeleton
- reserve composition accounting
- redeem burn skeleton
- USDC payout accounting
- fee separation tests
- balance delta tests
```

### Milestone 4: Venue / CPI Validation

```txt
- controlled adapter for tests
- Orca CPI feasibility
- Raydium compatibility review
- Surfpool mainnet-fork validation
- ApprovedRoute validation model
```

### Milestone 5: P0 End-to-End

```txt
- create market
- mint DTF with USDC
- reserve assets increase
- DTF token issued
- redeem DTF
- USDC returned
- fee accounting remains separated
- failure cases covered
```

---

## 15. Summary

Axis v1 Core should be implemented as a clean new contract system in the new `axis-core` repository.

The first phase should focus only on the on-chain reserve-backed DTF Core.

The core product model is:

```txt
USDC in
→ approved venue execution
→ reserve assets held by program-controlled vaults
→ Token-2022 DTF token minted
→ DTF burned on redeem
→ reserve assets unwound
→ USDC out
```

The current phase should avoid over-scoping into backend, database, frontend, Auction Program, ClearCorrection, or JIT liquidity.

Those areas remain important, but they should not block the first Axis Core implementation.

The next step is to review this RFC with engineering contributors, then convert the agreed P0 scope into implementation issues.

# Pre-Mainnet Validation Requirements

## 1. Overview

This document defines the pre-mainnet validation requirements for Axis v1.

Axis v1 does not treat public Devnet deployment as a mandatory validation step.

The goal of pre-mainnet validation is to prove that Axis v1 can safely operate as a reserve-backed DTF protocol before mainnet launch.

Pre-mainnet validation should prioritize deterministic local tests, fork-based integration tests, production venue integration validation, app contract integration checks, and guarded mainnet launch controls.

Public Devnet may be used opportunistically if it becomes useful, but it is not required for mainnet readiness.

## 2. Rationale

Public Devnet is not always representative of mainnet conditions.

Axis v1 depends on contract-level accounting, CPI execution, venue integration, reserve custody, creator fee behavior, and actual token balance delta verification.

If the public Devnet environment is unstable, incomplete, not representative, or not aligned with the deployed mainnet program, deploying to Devnet does not by itself provide meaningful readiness evidence.

Therefore, Axis v1 readiness must be based on validation evidence, not on whether the protocol has run on public Devnet for a fixed amount of time.

## 3. Validation Strategy

Axis v1 pre-mainnet validation should follow this strategy:

* deterministic unit tests
* local integration tests
* LiteSVM tests
* local validator tests
* mainnet-fork or cloned-account tests where applicable
* venue CPI integration tests
* app contract integration smoke tests
* guarded mainnet launch preparation
* documented readiness evidence

The validation path should prove the following:

* DTF markets can be created
* DTF tokens are backed by actual reserve assets
* mint uses real execution and actual balance deltas
* redeem uses real execution and actual output deltas
* creator fees are implemented and tested
* reserve accounting is correct
* NAV and supply accounting are consistent
* failure cases revert safely
* venue integration works before mainnet launch
* app transaction construction works against the new contract model
* launch controls are ready before mainnet exposure

## 4. Public Devnet Policy

Public Devnet is not part of the required Axis v1 validation path.

Public Devnet may be used only if it provides meaningful validation value.

Public Devnet must not be used as a substitute for deterministic tests, fork-based tests, venue CPI tests, or guarded launch controls.

The following must not be assumed:

* public Devnet liquidity exists
* public Devnet DEX pools are reliable
* public Devnet program deployments match mainnet binaries
* public Devnet execution behavior is representative of mainnet
* running on Devnet for a fixed number of days proves mainnet readiness

If Devnet is used, it should be treated as an optional demo or additional validation environment, not as the primary readiness gate.

## 5. Validation Layers

### 5.1 Unit Tests

Unit tests must validate isolated program logic.

Required coverage areas:

* DTF market initialization rules
* asset count validation
* weight sum validation
* duplicate asset rejection
* creator fee configuration
* protocol fee configuration
* fee ceiling validation
* NAV math
* reserve value math
* share supply math
* min allocation checks
* policy validation
* pause and emergency state transitions
* invalid authority rejection

### 5.2 Local Integration Tests

Local integration tests must validate end-to-end DTF lifecycle behavior in a controlled environment.

Required coverage areas:

* create DTF market
* initialize reserve accounts
* mint DTF with USDC or test USDC
* execute CPI path
* move real SPL tokens
* verify reserve balance deltas
* calculate actual added value
* mint DTF shares
* accrue creator fees
* redeem DTF shares
* unwind reserve assets
* verify USDC output balance delta
* burn or remove redeemed DTF shares
* enforce min_out
* revert failed execution

### 5.3 Fork-Based Tests

Fork-based tests should be used when validation depends on mainnet-like accounts, venue programs, token mints, or CPI behavior.

Fork-based tests should validate:

* production venue program IDs
* cloned token mints
* cloned pool accounts where applicable
* CPI account requirements
* real venue instruction construction
* compute usage
* account limit usage
* Token / Token-2022 behavior where applicable
* route constraints
* failure behavior

Fork-based tests are especially important for venue integrations.

### 5.4 Venue CPI Integration Tests

Axis v1 must validate at least two production venue integration paths before mainnet launch, so that Axis is not dependent on a single execution venue. See `05-swap-cpi-execution-requirements.md` (EXEC-015).

A venue is not considered Axis-ready only because an SDK can quote a route.

A venue is Axis-ready only when Axis Core can:

* validate the approved route
* execute the venue CPI
* move real tokens
* verify actual token balance deltas
* enforce min_out
* reject invalid accounts
* measure compute usage
* measure account usage
* handle failure cases safely

Required production venue paths:

* Orca Whirlpool (first production venue candidate)
* Raydium CPMM (fallback production venue candidate)

Later candidates (not required for the first mainnet launch gate):

* PumpSwap
* Raydium CLMM
* Meteora DLMM

The first two production venues do not need to be all venues.

However, at least two production-grade mint/redeem execution paths (Orca Whirlpool and Raydium CPMM) must be validated before mainnet launch.

### 5.5 App Contract Integration Tests

The app does not need to be fully redesigned as part of this document.

However, the app must be able to connect to the Axis v1 contract model.

App contract integration tests should validate:

* loading DTF market state
* loading reserve accounts
* loading target weights
* loading actual reserve balances
* loading NAV or approved pricing output
* constructing mint transactions
* constructing redeem transactions
* submitting transactions
* displaying transaction results
* displaying reserve-backed DTF state
* displaying creator fee information where relevant

Detailed UI and product interaction requirements should be defined in a separate app requirements document.

### 5.6 Guarded Mainnet Launch Controls

If Axis proceeds without mandatory public Devnet rollout, mainnet launch must be guarded.

Guarded launch controls should include:

* limited initial asset universe
* approved route registry
* per-asset execution policy
* per-transaction limits
* creator fee limits
* protocol fee limits
* emergency pause controls
* asset disable controls
* route disable controls
* monitoring
* known rollback or mitigation procedures
* documented launch checklist

Mainnet launch must not be treated as a general public launch until required validation evidence is available.

### 5.7 Separate Readiness Gates

Axis v1 readiness is divided into distinct gates:

```txt
Core mainnet readiness
  Reserve-backed DTF creation, mint, redeem, NAV, accounting, fees, and safety controls are validated.

Launch-day secondary-market surface readiness
  DTFs are discoverable for secondary use and any external liquidity is accurately labelled.

Production venue execution readiness
  Role A Orca Whirlpool and Raydium CPMM mint/redeem execution paths are validated.

Axis-controlled JIT liquidity spike readiness
  The Orca Role B technical spike produces feasibility and safety evidence. This is not a general launch blocker.

Production ClearCorrection readiness, if activated
  A market-specific Axis Auction Program configuration may activate only after its applicable readiness gates pass.
```

## 6. Requirements

### PMV-001: Public Devnet must not be required for mainnet readiness

Axis v1 must not depend on public Devnet deployment as a mandatory readiness gate.

Acceptance criteria:

* public Devnet deployment is optional
* readiness is based on validation evidence
* local tests are required
* fork-based tests are required where applicable
* venue CPI validation is required before mainnet launch
* guarded mainnet launch controls are required

### PMV-002: Pre-mainnet validation must prove the core DTF lifecycle

Pre-mainnet validation must prove that Axis Core can operate a reserve-backed DTF lifecycle.

Acceptance criteria:

* a DTF market can be created
* DTF market assets are validated
* target weights are validated
* reserve accounts are initialized
* user can mint DTF with USDC or test USDC
* execution path moves real SPL tokens
* reserve balances increase based on actual received assets
* DTF shares are minted based on actual added value
* user can redeem DTF shares
* reserve assets are unwound back to USDC or test USDC
* redeemed DTF shares are burned or otherwise removed from circulating supply

### PMV-003: Accounting must use actual balance deltas

Axis v1 accounting must use actual token balance deltas as protocol truth.

Acceptance criteria:

* pre-execution balances are recorded
* post-execution balances are recorded
* reserve asset deltas are calculated
* USDC output deltas are calculated
* quotes are not used as final accounting truth
* target weights are not treated as actual reserves
* UI estimates are not treated as protocol accounting truth

### PMV-004: Mint must be validated end-to-end

Mint validation must prove that USDC input creates reserve-backed DTF shares.

Acceptance criteria:

* user USDC input is validated
* target allocation is calculated
* min allocation checks are enforced
* approved execution path is used
* CPI execution moves real tokens
* actual reserve balance deltas are verified
* actual added value is calculated
* DTF shares are minted according to pre-trade NAV
* creator fee behavior is applied where relevant
* failed mint reverts the full transaction

### PMV-005: Redeem must be validated end-to-end

Redeem validation must prove that DTF shares can be redeemed into USDC output.

Acceptance criteria:

* user DTF input is validated
* redeem share is calculated
* pro-rata reserve asset amounts are calculated
* approved unwind path is used
* CPI execution moves real tokens
* actual USDC output delta is verified
* min_usdc_out is enforced
* user_usdc_out equals actual_usdc_received (redeem_fee_bps = 0)
* no creator or protocol fee accrues on redeem
* redeemed DTF shares are burned or otherwise removed from circulating supply
* failed redeem reverts the full transaction

### PMV-006: Creator fee behavior must be implemented and tested

Axis v1 must support creator fee behavior at the contract level before mainnet launch.

Acceptance criteria:

* DTF market stores creator or creator fee recipient
* creator fee configuration is validated
* creator fee bps is bounded
* creator fee accrual is tested
* creator fee claim or sweep path is defined
* creator and protocol fees accrue only on mint; no fee accrues on redeem
* fees are claim-based, not immediately transferred
* creator fee accounting does not break reserve accounting
* creator fee behavior is visible to app integration where relevant

### PMV-007: Role A venue integration must be validated before mainnet launch

Axis v1 must validate at least two production venue integration paths before mainnet launch (Orca Whirlpool and Raydium CPMM fallback). See `05-swap-cpi-execution-requirements.md` (EXEC-015, EXEC-016, EXEC-017).

Acceptance criteria:

* both production venue candidates (Orca Whirlpool and Raydium CPMM) are validated
* each venue program ID is verified
* route account requirements are documented per venue
* CPI instruction construction is tested per venue
* CPI execution moves real tokens
* pre/post token balance deltas are verified
* min_out is enforced
* invalid account failure is tested
* compute usage is measured per venue
* account usage is measured per venue
* venue-specific risks are documented
* controlled adapter tests are not sufficient for production venue readiness

### PMV-008: Failure cases must be tested

Pre-mainnet validation must prove that unsafe or invalid actions fail.

Acceptance criteria:

* min_out failure reverts
* disabled asset mint fails
* disabled route fails
* wrong input mint fails
* wrong output mint fails
* wrong reserve account fails
* wrong market account fails
* wrong authority fails
* invalid creator fee configuration fails
* invalid target weights fail
* invalid asset count fails
* paused market blocks restricted actions
* emergency controls behave as specified

### PMV-009: Execution must be all-or-nothing

Mint and redeem must not partially succeed.

Acceptance criteria:

* failed validation reverts the full transaction
* failed CPI execution reverts the full transaction
* failed min_out check reverts the full transaction
* DTF shares are not minted if reserve accounting fails
* DTF shares are not burned if redeem accounting fails
* creator fees are not accrued if the parent action fails
* reserve balances are not accepted without matching accounting state

### PMV-010: Reserve backing must be observable

Axis v1 must make reserve backing observable for validation, app integration, and partner review.

Acceptance criteria:

* reserve token accounts can be identified
* reserve balances can be read
* target weights can be read
* DTF supply can be read
* NAV or approved reserve value can be derived
* actual reserve backing can be shown for demo and review
* app integration can display reserve-backed DTF state

### PMV-011: App integration must validate contract connectivity

The app must be able to connect to the Axis v1 contract model before partner-facing demos or mainnet launch.

Acceptance criteria:

* app can load DTF market state
* app can load reserve balances
* app can load target weights
* app can load NAV or approved pricing output
* app can construct mint transaction
* app can construct redeem transaction
* app can display transaction results
* app can display creator fee information where relevant
* app can display reserve-backed state

Detailed UX and UI requirements are out of scope for this document.

### PMV-012: Mainnet binary and source must be traceable

Mainnet deployment must be traceable to the intended source code and build configuration.

Acceptance criteria:

* deployed program ID is recorded
* source commit is recorded
* build profile is recorded
* upgrade authority is recorded
* program data length is recorded
* deployed binary/source equivalence is checked where possible
* deployment notes are documented
* any mismatch between source and deployed program is documented

### PMV-013: Guarded mainnet launch controls must be ready

If public Devnet is not used as a required validation path, mainnet launch must use explicit guards.

Acceptance criteria:

* initial asset universe is limited
* approved routes are limited
* per-transaction limits are configured
* fee limits are configured
* emergency pause controls are available
* route disable controls are available
* asset disable controls are available
* monitoring plan is defined
* launch checklist is documented
* known rollback or mitigation procedures are documented

### PMV-014: Partner demo readiness must be supported by real validation evidence

Partner demos must be backed by contract-level evidence, not only mockups.

Acceptance criteria:

* contract GitHub repository can be shared
* core DTF lifecycle is documented
* mint/redeem behavior can be demonstrated
* reserve backing can be shown
* venue integration status can be explained
* transaction evidence or local/fork test evidence can be shared where appropriate
* partner-specific integration story can be explained

Partner demo requirements should be defined in a separate partner BD requirements document.

### PMV-015: Launch-day secondary-market surface must be validated before launch

The launch-day secondary-market surface is a required mainnet launch gate. It does not require production ClearCorrection or an active Axis-native JIT configuration for every DTF market.

Acceptance criteria:

```txt
- created DTF markets are discoverable
- DTF token mints are visible
- known external pool references can be displayed where available
- external liquidity is labelled accurately and distinctly from Axis-native auction-enabled liquidity
- no LVR-mitigation claim is made for an external public pool
- canonical market/share URL and partner or sponsor campaign metadata are available where the Axis-operated surface provides them
- a market that is architecturally compatible but inactive is not labelled auction/JIT enabled
```

### PMV-016: Axis-controlled JIT liquidity spike must be validated before native activation

Axis-controlled JIT liquidity requires separate technical-spike evidence before a market-specific native configuration can activate. This requirement is not a hard August launch blocker for the secondary-market surface.

Acceptance criteria:

```txt
- Orca DTF/USDC Whirlpool feasibility is evaluated
- controlled position ownership or authority feasibility is evaluated
- increase-liquidity, correction-swap, and decrease-liquidity ordering is tested
- account, tick-array, compute-unit, and transaction-size requirements are measured
- one-transaction settlement is evaluated first
- Jito fallback is evaluated only if the one-transaction path is infeasible
- wrong winner, wrong pool, stale NAV, replay, expiry, and failure rollback paths are tested
- auction revenue separation is tested
```

### PMV-017: Production ClearCorrection readiness is conditional on activation

Production ClearCorrection and the Axis Auction Program are required only for a market that Axis explicitly activates for Axis-controlled JIT liquidity. They are not required for the minimum August launch gate unless a later decision changes that scope.

Acceptance criteria for an activated market:

```txt
- pool availability, pricing freshness, route support, Auction Program support, account/compute feasibility, and safety validation pass
- ClearCorrection is winner-authorized, bounded, and separate from Axis Core reserve/NAV/accounting authority
- single-transaction settlement succeeds, or a validated Jito fallback provides ordered atomic execution and non-winner interception protection
- failed correction does not record successful payment or affect Axis Core reserve accounting
- auction revenue is excluded from DTF reserves, NAV, and mint/redeem fees
```

## 7. Required Test Scenarios

The following test scenarios must be covered before mainnet launch.

### 7.1 Market Creation

* create a valid 2-asset DTF market
* create a valid 3 to 5 asset DTF market where supported
* reject 1-asset DTF market
* reject more than 5 assets
* reject duplicate assets
* reject invalid weight sum
* reject weight below minimum
* reject disabled asset
* validate creator fee configuration
* validate reserve account ownership

### 7.2 Mint

* first mint with initial NAV
* subsequent mint with pre-trade NAV
* mint with exact target allocation
* mint with actual received amount different from quote
* mint with creator fee
* mint with protocol fee where applicable
* mint with min_out protection
* mint failure on disabled route
* mint failure on disabled asset
* mint failure on invalid reserve account
* mint failure on insufficient output
* mint failure on invalid pricing source

### 7.3 Redeem

* redeem DTF into USDC
* redeem partial DTF position
* redeem full DTF position
* redeem charges no explicit Axis fee (redeem_fee_bps = 0; user_usdc_out = actual_usdc_received)
* redeem accrues no creator or protocol fee
* redeem with min_usdc_out protection
* redeem failure on disabled route
* redeem failure on invalid reserve account
* redeem failure on insufficient output
* redeem behavior under paused or emergency state
* redeem behavior when mint is disabled but exit remains allowed where specified

### 7.4 Accounting

* actual reserve balance delta calculation
* actual USDC output delta calculation
* NAV calculation
* DTF supply calculation
* creator fee accounting
* protocol fee accounting
* reserve value calculation
* target weight vs actual reserve distinction
* mixed decimals normalization
* supply denominator consistency

### 7.5 Venue CPI

* valid CPI execution
* invalid venue program rejection
* invalid pool account rejection
* invalid input mint rejection
* invalid output mint rejection
* min_out failure
* compute usage measurement
* account usage measurement
* Token / Token-2022 behavior where applicable
* route-specific failure behavior

### 7.6 Admin and Safety

* pause restricted actions
* route disable
* asset disable
* fee update validation
* creator fee bounds
* protocol fee bounds
* emergency controls
* authority validation
* upgrade authority documentation
* invalid authority rejection

### 7.7 App Contract Integration

* load DTF market
* load reserve balances
* load target weights
* load NAV or reserve value
* construct mint transaction
* construct redeem transaction
* display transaction success
* display transaction failure
* display reserve-backed state
* display creator fee information where relevant

### 7.8 Launch-Day Secondary-Market Surface

* created DTF market is discoverable
* DTF token mint is visible
* known external DTF/USDC pool reference is displayed where available
* external liquidity label is distinct from Axis-native auction-enabled liquidity
* no false Axis LVR-mitigation claim is displayed for an external pool
* canonical market/share URL and partner/sponsor metadata are available where applicable
* architecturally compatible but inactive status is displayed accurately

### 7.9 Axis-Controlled JIT / ClearCorrection Spike

These are proposed activation tests, not minimum launch-day secondary-market tests.

* Orca DTF/USDC Whirlpool feasibility
* controlled position/authority feasibility
* increase/swap/decrease ordering
* account, tick-array, compute, and transaction-size measurement
* one-transaction feasibility
* Jito fallback evaluation only if required
* wrong winner, wrong pool, stale NAV, replay, expiry, and failure rollback rejection
* auction revenue separation from reserves, NAV, and mint/redeem fees

## 8. Partner Integration Context

Axis v1 partner demos should support the following integration narratives.

### 8.1 Titan

Axis issues reserve-backed DTF markets.

Titan may become a distribution or routing surface for Axis-issued DTF exposure.

The intended partner story is:

* Axis handles DTF issuance
* Axis owns the reserve-backed DTF protocol
* Titan can route or surface swaps into Axis-issued DTF exposure
* users may eventually obtain DTF exposure from Titan UI
* Axis and Titan can collaborate on distribution and liquidity access

Titan integration is not the same as Axis mint/redeem venue integration.

Titan is a distribution and routing partner, not the required core execution venue for Axis mint/redeem.

### 8.2 Orca

Orca may be used as a production execution venue for DTF mint/redeem flows.

The intended partner story is:

* Axis uses Orca CPI in DTF execution where appropriate
* Axis can show the integration code
* Axis can ask Orca to review whether the design is correct
* the initial ask is technical validation and design feedback
* deeper collaboration can follow after the integration path is proven

### 8.3 Other Solana Projects

Axis can use the validated contract and partner demo flows to build relationships with other Solana projects.

Potential collaboration areas include:

* DTF issuance around partner ecosystems
* distribution of partner-themed DTF exposure
* routing integrations
* liquidity integrations
* future leverage or margin products on DTF markets
* co-marketing around reserve-backed narrative markets

## 9. Mainnet Launch Gate

Axis v1 must not move to general public mainnet launch until the mainnet launch gate is satisfied.

Minimum launch gate:

* local tests pass
* integration tests pass
* fork-based tests pass where applicable
* at least two production venue integration paths are validated (Orca Whirlpool and Raydium CPMM fallback)
* controlled adapter tests alone are not treated as production venue readiness
* core create/mint/redeem lifecycle is validated
* actual balance delta accounting is validated
* reserve backing is observable
* creator fee behavior is implemented and tested
* app contract integration is tested
* launch-day secondary-market surface is validated, including accurate external-liquidity labels
* guarded launch controls are configured
* deployment source and binary traceability are documented
* known blockers are resolved or explicitly accepted
* launch checklist is completed

Production ClearCorrection / Axis Auction Program activation is not a minimum August launch condition. It becomes a separate production gate only for an explicitly activated Axis-controlled JIT liquidity configuration.

## 10. Out of Scope

The following are out of scope for this document:

* full frontend redesign
* final UI component requirements
* final UX copy
* full app page architecture
* full partner BD deck
* Titan production integration
* leverage trading implementation
* generalized routing aggregator
* full 500-asset production universe
* full permissionless route creation
* public Devnet deployment as a required milestone
* production ClearCorrection / full Axis Auction Program activation as a minimum August launch requirement

These should be handled in separate documents where needed.

## 11. Open Questions

The following decisions remain open:

* production venue candidates beyond the first two (Orca Whirlpool and Raydium CPMM are confirmed; see 05-swap-cpi-execution-requirements.md)
* creator fee claim or sweep mechanism (fee bps and split are confirmed; see 13-fee-model-requirements.md)
* initial mainnet asset universe
* initial guarded launch limits
* initial approved route set
* whether initial launch is invite-only, partner-only, or public
* exact app integration milestone
* reserve/NAV display source for app
* monitoring and alerting setup
* upgrade authority and governance process
* audit timing
* partner demo sequence

## 12. Issue Candidates

* Rename Devnet rollout requirements to pre-mainnet validation requirements
* Define local test matrix for Axis v1
* Define LiteSVM test matrix
* Define mainnet-fork test matrix
* Implement DTF market creation validation tests
* Implement mint lifecycle integration tests
* Implement redeem lifecycle integration tests
* Implement actual balance delta accounting tests
* Implement creator fee tests
* Implement failure case tests
* Implement all-or-nothing execution tests
* Validate Orca Whirlpool and Raydium CPMM production venue CPI paths
* Measure venue CPI compute usage (per venue)
* Measure venue CPI account usage (per venue)
* Define guarded mainnet launch checklist
* Document deployment source and binary traceability
* Define app contract integration smoke tests
* Define partner demo evidence requirements
* Define mainnet launch gate checklist
* Validate launch-day secondary-market surface and external-liquidity labels
* Run Orca Axis-controlled JIT liquidity technical spike
* Define conditional production ClearCorrection readiness checklist

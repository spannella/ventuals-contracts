# File-by-file Guide

This guide walks through the purpose of each maintained file in the repository.
It focuses on the contract sources, scripts, tests, and configuration that make
up the Ventuals vHYPE system.

## Repository root

- **README.md** — Quick project overview, high-level protocol summary, and basic
  setup/testing commands, plus a pointer to the more detailed getting-started
  guide.【F:README.md†L1-L56】
- **foundry.toml** — Foundry configuration for compiler settings, remappings,
  network endpoints, and an additional `deploy` profile tailored for smaller
  bytecode deployments.【F:foundry.toml†L1-L32】
- **foundry.lock** — Locks exact git revisions of Foundry dependencies such as
  forge-std, OpenZeppelin contracts, and Solady so builds are reproducible.【F:foundry.lock†L1-L25】
- **remappings.txt** — Remapping entries used by Foundry to resolve OpenZeppelin
  and Solady imports from the `lib/` directory tree.【F:remappings.txt†L1-L3】
- **package.json** — Node.js metadata and dependencies for the helper scripts,
  including the `deposit` and `transfer-to-core` npm run targets.【F:package.json†L1-L13】
- **gencoverage.sh** — Convenience shell script that runs `forge coverage`,
  generates an HTML report via `genhtml`, and opens the report locally.【F:gencoverage.sh†L1-L16】
- **grind-salts.sh** — Script to grind CREATE2 salts for implementations and
  proxies, assembling `.env.salts` for deterministic deployments using Forge
  and Cast utilities.【F:grind-salts.sh†L1-L215】

## docs/

- **docs/ARCHITECTURE.md** — Protocol architecture overview describing the vault,
  manager, token, and role registry, plus the deposit/withdraw lifecycle and
  HyperEVM/HyperCore timing constraints.【F:docs/ARCHITECTURE.md†L1-L211】
- **docs/GETTING_STARTED.md** — Step-by-step setup guide covering dependencies,
  build/test commands, scripts, and the `deploy` profile for size-constrained
  deployments.【F:docs/GETTING_STARTED.md†L1-L90】
- **docs/FILE_GUIDES.md** — This file-by-file reference guide.

## src/

### Core contracts

- **src/Base.sol** — Shared upgradeable base class that wires the RoleRegistry,
  centralizes pause checks, and provides `onlyOwner`, `onlyManager`, and
  `onlyOperator` access modifiers used across the system.【F:src/Base.sol†L1-L75】
- **src/RoleRegistry.sol** — Central access-control registry that manages
  MANAGER/OPERATOR roles, pause state for contracts, and UUPS upgrade
  authorization under an Ownable2Step owner.【F:src/RoleRegistry.sol†L1-L79】
- **src/StakingVault.sol** — Vault contract that holds and moves HYPE, delegates
  and undelegates via CoreWriter, guards precompile reads with one-block delays,
  and enforces validator whitelisting for staking actions.【F:src/StakingVault.sol†L1-L229】
- **src/StakingVaultManager.sol** — Main manager that handles deposits, queued
  withdrawals, batch processing, exchange rates, and emergency operations, while
  coordinating with StakingVault and vHYPE mint/burn logic.【F:src/StakingVaultManager.sol†L1-L792】
- **src/VHYPE.sol** — Upgradeable ERC20 token implementation that can only be
  minted/burned by the MANAGER role and is pause-aware via Base modifiers.【F:src/VHYPE.sol†L1-L33】

### Interfaces

- **src/interfaces/ICoreWriter.sol** — Minimal interface for the HyperCore
  CoreWriter contract, exposing `sendRawAction` for low-level action dispatching.【F:src/interfaces/ICoreWriter.sol†L1-L6】
- **src/interfaces/IStakingVault.sol** — Interface defining vault errors,
  events, staking/unstaking calls, HyperCore interactions, and validator
  management functions used by managers and tests.【F:src/interfaces/IStakingVault.sol†L1-L106】
- **src/interfaces/IStakingVaultManager.sol** — Manager interface describing
  errors, events, batch/withdraw structures, and the public methods exposed to
  depositors and operators.【F:src/interfaces/IStakingVaultManager.sol†L1-L247】

### Libraries

- **src/libraries/Converters.sol** — Utility conversions between 8- and
  18-decimal HYPE representations, plus a helper to strip unsafe precision when
  crossing HyperEVM/HyperCore boundaries.【F:src/libraries/Converters.sol†L1-L20】
- **src/libraries/CoreWriterLibrary.sol** — Encodes and forwards HyperCore
  actions (delegate, deposit, withdraw, spot send, add API wallet) using the
  CoreWriter precompile endpoint and action IDs.【F:src/libraries/CoreWriterLibrary.sol†L1-L75】
- **src/libraries/L1ReadLibrary.sol** — Precompile read helpers and data
  structures for HyperCore state (spot balances, delegations, token info, etc.).【F:src/libraries/L1ReadLibrary.sol†L1-L240】
- **src/libraries/StructuredLinkedList.sol** — Linked list implementation used
  for the withdrawal queue, providing node traversal, insert/remove, and list
  management helpers.【F:src/libraries/StructuredLinkedList.sol†L1-L240】

## script/

- **script/DeployContracts.s.sol** — Foundry deployment script that loads salts
  and expected addresses, deploys implementations and proxies with CREATE2,
  configures validators and vault parameters, and grants MANAGER_ROLE to the
  StakingVaultManager proxy.【F:script/DeployContracts.s.sol†L1-L362】
- **script/RoleRegistryGrantRole.s.sol** — Script to grant a role (`manager` or
  `operator`) to an account via RoleRegistry using environment variables for
  target addresses and role selection.【F:script/RoleRegistryGrantRole.s.sol†L1-L49】
- **script/RoleRegistryHasRole.s.sol** — Script that checks whether an account
  has a specified role in the RoleRegistry, logging the result via `console`.【F:script/RoleRegistryHasRole.s.sol†L1-L40】
- **script/UpgradeStakingVault.s.sol** — Upgrade helper that deploys a new
  StakingVault implementation (with correct HYPE token ID) and upgrades the
  proxy via `upgradeToAndCall`.【F:script/UpgradeStakingVault.s.sol†L1-L25】
- **script/UpgradeStakingVaultManager.s.sol** — Upgrade helper that deploys a
  new StakingVaultManager implementation and upgrades the proxy via
  `upgradeToAndCall`.【F:script/UpgradeStakingVaultManager.s.sol†L1-L20】

## scripts/

- **scripts/README.md** — Detailed usage docs for the Node.js and bash helper
  scripts, including required environment variables, expected outputs, and
  example commands.【F:scripts/README.md†L1-L160】
- **scripts/call-deposit.js** — Node.js script that loads env config, selects
  mainnet/testnet RPC, checks balance, and calls the manager’s `deposit()` with
  a configurable deposit amount and gas limit.【F:scripts/call-deposit.js†L1-L112】
- **scripts/call-transfer-to-core.js** — Node.js script that calls
  `transferToCoreAndDelegate` (with optional amount argument), choosing RPC by
  environment and printing transaction details and balances.【F:scripts/call-transfer-to-core.js†L1-L175】
- **scripts/addApiWallet.sh** — Bash script that reads environment variables and
  invokes `addApiWallet` on the StakingVault via `cast send` on testnet RPC.【F:scripts/addApiWallet.sh†L1-L48】
- **scripts/deposit.sh** — Bash script that deposits a user-specified amount of
  HYPE into the manager contract using `cast send` and the testnet RPC URL.【F:scripts/deposit.sh†L1-L41】
- **scripts/transferToCoreAndDelegate.sh** — Bash script that calls
  `transferToCoreAndDelegate(uint256)` with a user-provided wei amount on
  the testnet RPC endpoint.【F:scripts/transferToCoreAndDelegate.sh†L1-L40】

## test/

- **test/HyperCoreSimulator.sol** — Test harness that mocks HyperCore state,
  CoreWriter behavior, and precompile contracts, plus helpers for time/block
  simulation and CoreWriter call assertions.【F:test/HyperCoreSimulator.sol†L1-L82】
- **test/RoleRegistry.t.sol** — Tests RoleRegistry initialization, role grant
  and revoke behavior, pause/unpause, ownership transfer, and upgrade
  authorization workflows.【F:test/RoleRegistry.t.sol†L1-L414】
- **test/StakingVault.t.sol** — Comprehensive StakingVault tests covering stake
  and unstake flows, redelegation, spot sends, validator management, pause
  behavior, and upgrade mechanics with HyperCore mocks.【F:test/StakingVault.t.sol†L1-L1029】
- **test/StakingVaultManager.t.sol** — StakingVaultManager tests for
  initialization, deposit/withdraw flows, batch processing, exchange rate
  behavior, and emergency operations using the HyperCore simulator.【F:test/StakingVaultManager.t.sol†L1-L240】
- **test/VHYPE.t.sol** — vHYPE token tests verifying mint/burn permissions,
  allowance-based burns, transfers, pause behavior, and upgrade authorization.【F:test/VHYPE.t.sol†L1-L335】

### test/mocks/

- **test/mocks/Constants.sol** — Shared constants for mock addresses and token
  IDs used by the HyperCore simulator and precompile mocks.【F:test/mocks/Constants.sol†L1-L13】
- **test/mocks/MockCoreWriter.sol** — Routes CoreWriter calls to the
  MockHyperCoreState for recording simulated HyperCore actions.【F:test/mocks/MockCoreWriter.sol†L1-L12】
- **test/mocks/MockHypeSystemContract.sol** — Receives value transfers and
  records them in MockHyperCoreState to emulate HyperEVM → HyperCore transfers.【F:test/mocks/MockHypeSystemContract.sol†L1-L12】
- **test/mocks/MockHyperCoreState.sol** — Core mock state machine that tracks
  delegations, balances, pending withdrawals, and records CoreWriter actions for
  tests to assert against.【F:test/mocks/MockHyperCoreState.sol†L1-L200】
- **test/mocks/MockPrecompileCoreUserExists.sol** — Precompile mock returning
  whether a HyperCore user exists for the requested address.【F:test/mocks/MockPrecompileCoreUserExists.sol†L1-L12】
- **test/mocks/MockPrecompileDelegations.sol** — Precompile mock returning
  delegations for a user from MockHyperCoreState.【F:test/mocks/MockPrecompileDelegations.sol†L1-L12】
- **test/mocks/MockPrecompileDelegatorSummary.sol** — Precompile mock returning
  delegator summary data for a user from MockHyperCoreState.【F:test/mocks/MockPrecompileDelegatorSummary.sol†L1-L12】
- **test/mocks/MockPrecompileSpotBalance.sol** — Precompile mock returning
  spot balances for a user/token pair from MockHyperCoreState.【F:test/mocks/MockPrecompileSpotBalance.sol†L1-L12】

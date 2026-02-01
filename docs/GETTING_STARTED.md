# Getting Started

This repository contains the Solidity contracts, Foundry configuration, and helper scripts
for the Ventuals vHYPE liquid staking system. Use the steps below to build, test, and run
scripts locally.

## Prerequisites

- **Foundry** for building, testing, and running Solidity scripts.
- **Node.js (LTS)** if you want to run the JavaScript helper scripts under `scripts/`.
- **Git submodules** enabled (OpenZeppelin contracts are vendored as submodules).

Install Foundry:

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
```

## Clone and install

```bash
git clone https://github.com/ventuals/ventuals-contracts.git
cd ventuals-contracts

git submodule update --init --recursive
forge build
```

If you plan to use the Node scripts, install dependencies:

```bash
npm install
```

## Build and test

```bash
forge build
forge test
```

To generate a coverage report:

```bash
forge coverage
```

### Running tests in Codex

Codex runs in a non-interactive environment, so use the same Foundry commands
that you would run locally. From the repository root:

```bash
forge build
forge test
```

If you need the size-focused profile (e.g., when working with large contracts),
run:

```bash
FOUNDRY_PROFILE=deploy forge build
FOUNDRY_PROFILE=deploy forge test
```

To generate coverage within Codex:

```bash
forge coverage
```

## Configure environment variables

Some scripts expect a local `.env` file (for example, RPC URLs or private keys).
Check `scripts/README.md` for the exact variables each script expects before running
any transaction scripts.

## Run Foundry scripts

Foundry scripts live under `script/` and can be executed with `forge script`. For example:

```bash
forge script script/DeployContracts.s.sol --rpc-url <RPC_URL> --broadcast
```

Adjust the RPC URL and broadcast settings for your target network.

### Deploy profile for size-constrained networks

If deployments fail due to contract size limits (for example on testnet), use the
`deploy` profile in `foundry.toml`, which favors smaller bytecode:

```bash
FOUNDRY_PROFILE=deploy forge build
FOUNDRY_PROFILE=deploy forge script script/DeployContracts.s.sol --rpc-url <RPC_URL> --broadcast
```

### Enhanced deploy guide for large Hyperliquid dapps

If you are deploying a larger app to Hyperliquid and hit contract size limits,
work through the following checklist.

1. **Use the size-focused deploy profile** (IR + low optimizer runs) so bytecode
   stays as small as possible:

   ```bash
   FOUNDRY_PROFILE=deploy forge build
   ```

2. **Measure bytecode size before deployment** to see which contracts are
   approaching the limit:

   ```bash
   forge inspect <ContractName> bytecode | wc -c
   ```

   The output is the hex string length (characters). Divide by 2 to get bytes.

3. **Split responsibilities across contracts** when a single contract grows too
   large. Move optional or admin-only flows into smaller helper contracts or
   libraries. This repository already separates the vault, manager, and token
   into distinct UUPS proxies to keep implementations smaller.

4. **Move reusable logic into libraries** (or separate contracts) so the logic
   is shared instead of duplicated across multiple implementations.

5. **Limit on-chain constants and unused code paths** by removing features that
   are not required for initial deployment, and deferring them to upgrades once
   the system is live.

6. **Deploy with CREATE2 or proxies** so upgrades do not require re-deploying
   the full system. The `DeployContracts.s.sol` script already handles CREATE2
   salts and proxy deployment for you.

7. **Rebuild and redeploy after each reduction** to confirm bytecode changes and
   avoid surprises during the final broadcast.

## Run Node/Bash scripts

Helper scripts that interact with deployed contracts are in `scripts/`. Examples:

```bash
npm run deposit
npm run transfer-to-core
```

See `scripts/README.md` for details and required environment variables.

## Next steps

- Review the protocol overview in `docs/ARCHITECTURE.md`.
- Inspect contract sources in `src/` to understand each module.
- Run tests before making changes to confirm a clean baseline.

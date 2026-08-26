<div align="center">

# CleverCon

**The payment rail AI agents spend through on Stellar. Bounded by default, private by design.**

[![CI](https://github.com/clevercon-protocol/clevercon/actions/workflows/ci.yml/badge.svg)](https://github.com/clevercon-protocol/clevercon/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Network](https://img.shields.io/badge/Network-Stellar%20Testnet-7B2FFF)](https://stellar.expert/explorer/testnet)
[![CleverVault](https://img.shields.io/badge/CleverVault-Deployed-00C853)](https://stellar.expert/explorer/testnet/contract/CC4QX7ZVME7PO25GELU5VIM6BOSU7UBNJF56D46VMGBWQBBFQVIXYRZO)

[Architecture](docs/architecture.md) · [Roadmap](ROADMAP.md) · [Contributing](CONTRIBUTING.md)

</div>

## What it is

CleverCon is payment infrastructure for AI agents on Stellar. It lets you hand an agent a budget it can spend on its own, while a smart contract holds the money and enforces the rules. The agent can pay for the data, compute, and services it needs to do real work, but it never holds your funds and never spends outside what you allowed.

You deposit USDC into CleverVault, a non-custodial Soroban contract, and set a spending budget. The agent pays for services in USDC as it works, the contract caps the total at what you approved, and unused budget is refunded. The operator never touches your money, and every payment is a real Stellar transaction.

The direction the project is built toward is privacy. Most agent-payment tools put your budget, your approved payees, and every payment on a public ledger for anyone to read. CleverCon is designed so the enforcement stays on-chain while the rules stay yours: a spending policy the contract checks without revealing it, and a proof that spending followed the policy without exposing the amounts or counterparties. That work builds on the zero-knowledge engine that already runs on Stellar testnet as [CipherMit](https://github.com/Bosun-Josh121/ciphermit).

## How it works

1. Connect a wallet and deposit USDC into CleverVault.
2. Describe what you want done and set a budget.
3. An orchestrator plans the work and pays specialist services in USDC as each step completes, always within your budget.
4. The vault caps spending at the budget and refunds the rest. You can withdraw anytime.

The full fund-flow sequence and trust model are in [docs/architecture.md](docs/architecture.md).

## What runs today

- **CleverVault**, a non-custodial Soroban contract on testnet: deposits, budget locking, per-step release, refunds, multi-asset support, and admin controls, with a 100+ case test suite.
- **Direct on-chain interaction from the browser**: connect a wallet, add a USDC trustline, deposit, check balance, and withdraw, all signed in your wallet and settled against the contract with no server in the middle.
- **Orchestrator and open registry**: an orchestrator that plans work and hires specialist services, and a registry any HTTP service with a Stellar wallet and x402 or MPP support can join.
- **x402 and MPP payments** to specialist services.
- Placed 2nd in the Stellar Agents hackathon.

## Where it is going

- **Private spending policies**: a spending rule the contract enforces on every payment without making the rule public, using zero-knowledge proofs, built on the CipherMit engine.
- **Provable compliance**: a proof that all spending followed the policy, shareable with an auditor or partner without revealing the policy.
- **An on-chain registry**, a **Stellar MCP server** so any agent can pay for services under a policy, and a **reusable SDK**.

## Project structure

```
clevercon/
├── contracts/
│   ├── agent-vault/           CleverVault, the on-chain USDC treasury (Soroban/Rust)
│   └── budget-guardian/       earlier budget-tracking contract (legacy, unused)
├── packages/
│   ├── common/                shared TypeScript types, constants, wallet helpers
│   ├── registry/              service discovery and reputation API
│   ├── orchestrator/          planner, executor, vault client, WebSocket hub
│   ├── dashboard/             React 19 + Vite + Tailwind frontend
│   └── agents/                reference specialist services (oracle, web-intel, analysis, reporter)
├── scripts/                   setup, wallet, and lifecycle scripts
└── docs/                      architecture and development docs
```

## Tech stack

| Layer | Technology |
|---|---|
| Smart contract | Rust / Soroban (CleverVault) |
| Zero-knowledge (direction) | RISC Zero and Noir circuits, verified on-chain, from the CipherMit engine |
| Frontend | React 19, Vite, Tailwind CSS, direct Soroban and Horizon calls |
| Backend | Node.js 20, Express, TypeScript (npm workspaces) |
| Payments | `@x402/express`, `@x402/stellar`, `@stellar/mpp` |
| Wallets | `@creit.tech/stellar-wallets-kit` (Freighter, xBull, Albedo, LOBSTR, Rabet) |
| Chain access | `@stellar/stellar-sdk`, Horizon, Soroban RPC |

## Quick start

### Prerequisites

- Node.js 20+ (see `.nvmrc`)
- Freighter (or another supported wallet) set to testnet
- An Anthropic API key, only if you run the orchestrator locally

### Install and run the dashboard

```bash
git clone https://github.com/clevercon-protocol/clevercon.git
cd clevercon
npm install
cd packages/dashboard
npm run dev
```

The dashboard talks to the contract directly, so wallet connect, the USDC trustline, and vault deposit, balance, and withdraw work on testnet without any backend. To run the full stack (orchestrator, registry, and the reference services), see [docs/development.md](docs/development.md).

## Deploying the dashboard (Vercel)

The dashboard is a static site and deploys on Vercel free:

1. Create a Vercel project from this repo.
2. Set the Root Directory to `packages/dashboard`.
3. Add the env var `VITE_BACKEND_ENABLED=false` for a standalone build (wallet and contract interactions stay real; orchestrator-backed features show a placeholder).
4. Build and output settings come from `packages/dashboard/vercel.json`.

To point the dashboard at a running backend later, set `VITE_API_URL` and `VITE_WS_URL` and set `VITE_BACKEND_ENABLED=true`.

## Deploying the CleverVault contract

Requires Rust and `stellar-cli` 25+:

```bash
cd contracts/agent-vault && ./deploy.sh
```

This builds to WASM, deploys, initializes, runs a smoke test, and writes `AGENT_VAULT_CONTRACT_ID` to `.env`.

## Deployments

| Component | Network | Address |
|---|---|---|
| CleverVault | Stellar Testnet | [`CC4QX7ZV...QVIXYRZO`](https://stellar.expert/explorer/testnet/contract/CC4QX7ZVME7PO25GELU5VIM6BOSU7UBNJF56D46VMGBWQBBFQVIXYRZO) |
| USDC (SAC) | Stellar Testnet | [`CBIELTK6...HMXQDAMA`](https://stellar.expert/explorer/testnet/contract/CBIELTK6YBZJU5UP2WWQEUCYKLPU6AUNZ2BQ4WWFEIE3USCIHMXQDAMA) |

## Documentation

- [Architecture](docs/architecture.md)
- [Development guide](docs/development.md)
- [Roadmap](ROADMAP.md)
- [Changelog](CHANGELOG.md)
- [Security policy](SECURITY.md)
- [Contributing](CONTRIBUTING.md)

## License

MIT. See [LICENSE](LICENSE).

# Isolate — Trust as a Service for Agricultural Trade

[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](./LICENSE)
[![Stellar](https://img.shields.io/badge/Network-Stellar-black?style=for-the-badge&logo=stellar)](https://stellar.org)
[![Soroban](https://img.shields.io/badge/Contracts-Soroban%20(Rust)-orange?style=for-the-badge)](https://soroban.stellar.org)
[![CI](https://github.com/ikufidelade1-cpu/isolate/actions/workflows/ci.yml/badge.svg)](https://github.com/ikufidelade1-cpu/isolate/actions)

---

## What Is Isolate?

Isolate is a decentralized escrow protocol built on the Stellar blockchain that eliminates the trust gap in agricultural commodity trading. When a buyer in Lagos wants to purchase grain from a farmer in Kano, neither party wants to "send first." Isolate solves this with a programmable, neutral vault — powered by Soroban smart contracts — that holds funds securely until delivery is verified.

No intermediary. No bank. No risk of sending first. Just smart contracts enforcing fair trade.

---

## The Problem

Agricultural trade across regions is broken by a fundamental trust gap:
- Sellers fear sending goods without payment
- Buyers fear paying without receiving goods
- Transit damage and theft create disputes with no resolution mechanism
- Currency volatility erodes the value of agreed prices by delivery time

Isolate addresses every one of these with on-chain primitives.

---

## Key Features

| Feature | Description |
|---------|-------------|
| **Smart Escrow** | Funds locked in Soroban contract vault; released only on verified delivery |
| **Dynamic Loss Sharing** | Negotiable risk ratios (50/50, 70/30, etc.) encoded into every trade |
| **Proof of Delivery (PoD)** | Video-based delivery verification stored on IPFS via Pinata |
| **Volatility Protection** | Stellar Path Payments let users pay in NGN while locking value in cNGN |
| **Dispute Resolution** | Mediator quorum system for contested trades |
| **Automated Settlement** | 1% platform fee deducted automatically on successful completion |
| **Multi-Party Roles** | Buyer, seller, driver, and mediator roles enforced on-chain |
| **Deadline Enforcement** | Configurable delivery deadlines with automatic refund on expiry |

---

## How It Works

```
Buyer creates trade → funds locked in Soroban vault
        │
        ▼
Seller confirms & dispatches goods
        │
        ▼
Driver picks up → Proof-of-Delivery video captured
        │
        ▼
Buyer verifies delivery on-chain
        │
        ├── Verified → funds released to seller (minus 1% fee)
        └── Disputed → mediator quorum reviews PoD evidence → verdict executes
```

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    User Layer                       │
│   Buyer · Seller · Driver · Mediator                │
│   Freighter / Albedo Wallets                        │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────┐
│               Frontend Layer                        │
│   Next.js App Router · Stellar SDK · Supabase       │
└──────────┬─────────────────────────┬────────────────┘
           │                         │
┌──────────▼──────────┐   ┌──────────▼──────────────┐
│  Blockchain Layer   │   │      Backend Layer       │
│  Stellar Network    │   │  Node.js · Express       │
│  Soroban Contracts  │   │  REST API · OpenAPI      │
│  Horizon API        │   │  Supabase · Pinata IPFS  │
└─────────────────────┘   └──────────────────────────┘
```

For full architecture documentation see [docs/architecture.md](docs/architecture.md).

---

## Repository Layout

```
isolate/
├── contracts/              # Soroban smart contracts (Rust)
│   └── amana_escrow/
│       └── src/
│           ├── lib.rs       # Contract entry point
│           ├── escrow.rs    # Trade lifecycle
│           ├── delivery.rs  # PoD verification
│           ├── dispute.rs   # Mediator quorum
│           └── settlement.rs
├── frontend/               # Next.js App Router
├── backend/                # Node.js REST API + event indexer
├── mobile/                 # Expo React Native
├── shared-test-fixtures/   # Shared test helpers
├── scripts/                # Build, deploy, codegen scripts
├── docs/                   # Full documentation
└── package.json            # Monorepo root (pnpm)
```

---

## Smart Contract API

### Trade Lifecycle
```rust
create_trade(buyer, seller, amount, asset, deadline, loss_ratio) -> TradeId
confirm_trade(trade_id, seller)
cancel_trade(trade_id, caller)
get_trade(trade_id) -> Trade
```

### Delivery & Verification
```rust
submit_delivery(trade_id, driver, ipfs_cid)
verify_delivery(trade_id, buyer)
reject_delivery(trade_id, buyer, reason)
get_delivery_proof(trade_id) -> DeliveryProof
```

### Dispute Resolution
```rust
raise_dispute(trade_id, caller, evidence_cid)
submit_mediator_vote(trade_id, mediator, verdict)
resolve_dispute(trade_id)
get_dispute(trade_id) -> Dispute
```

### Settlement
```rust
release_funds(trade_id)
refund_buyer(trade_id)
claim_platform_fee(trade_id)
```

### Emergency
```rust
pause_trade(trade_id, admin)
unpause_trade(trade_id, admin)
emergency_refund(trade_id, admin)
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Smart Contracts | Rust · Soroban SDK |
| Frontend | Next.js 14 (App Router) · TypeScript · Tailwind CSS · Stellar SDK |
| Mobile | Expo · React Native · TypeScript |
| Backend | Node.js · Express · TypeScript · OpenAPI/Swagger |
| Storage | Supabase (off-chain metadata) · Pinata IPFS (PoD video) |
| Blockchain | Stellar Network · Horizon API · Soroban RPC |
| Observability | OpenTelemetry · distributed tracing · correlation IDs |
| Build | pnpm · Turborepo |
| Testing | Cargo test · Jest · Playwright |
| CI/CD | GitHub Actions |

---

## Quick Start

```bash
# Clone
git clone https://github.com/ikufidelade1-cpu/isolate.git
cd isolate

# Install dependencies
pnpm install

# Contracts
cd contracts/amana_escrow && cargo build

# Frontend
cd frontend && cp .env.example .env.local && pnpm dev

# Backend
cd backend && cp .env.example .env && pnpm dev

# Mobile
cd mobile && cp .env.example .env.local && pnpm start
```

See [docs/local-development-setup.md](docs/local-development-setup.md) for the full environment guide.

---

## Supported Assets

| Asset | Type | Use |
|-------|------|-----|
| XLM | Native | Default escrow asset |
| cNGN | SEP-41 Token | Nigerian Naira stablecoin |
| USDC | SEP-41 Token | USD stablecoin for cross-border |
| Custom SEP-41 | Token | Any Stellar-issued asset |

---

## Supported Wallets

| Wallet | Web | Mobile |
|--------|-----|--------|
| Freighter | ✅ | ✅ |
| Albedo | ✅ | ❌ |
| Lobstr | ✅ | ✅ |
| WalletConnect | ✅ | ✅ |

---

## Documentation

| Document | Description |
|----------|-------------|
| [Architecture](docs/architecture.md) | System design and component interactions |
| [Design Flows](docs/design-flows.md) | End-to-end trade and dispute flows |
| [User Guide](docs/user-guide.md) | Step-by-step guide for all roles |
| [Roadmap](docs/roadmap.md) | Versioned feature roadmap |
| [Contributing](CONTRIBUTING.md) | How to contribute |
| [API Reference](backend/src/docs/openapi.yaml) | Full REST API spec |
| [Contract Errors](docs/contract-error-codes.md) | Soroban error code reference |
| [Security](SECURITY.md) | Responsible disclosure policy |

---

## CI Gates

All pull requests must pass:
- Rust contract build + clippy + tests
- Frontend lint + typecheck + tests
- Backend lint + typecheck + tests + schema check
- Secret scanning (gitleaks)
- Dependency audit

---

## Roadmap Summary

- **v1.0** — Core escrow (create, confirm, verify, release) on Testnet
- **v1.1** — Dispute resolution + mediator quorum
- **v2.0** — Proof-of-Delivery IPFS integration + mobile app
- **v2.1** — cNGN/USDC multi-asset support + path payments
- **v3.0** — Reputation system + trade history credit scoring
- **v4.0** — Fiat on/off ramps + regional expansion

See [docs/roadmap.md](docs/roadmap.md) for full detail.

---

## Contributing

Issues labelled `good-first-issue` are great starting points. Funded issues are labelled `wave-ready`.

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full guide.

---

## License

MIT © [ikufidelade1-cpu](https://github.com/ikufidelade1-cpu)

---

*Building trust infrastructure for agricultural trade on Stellar.*

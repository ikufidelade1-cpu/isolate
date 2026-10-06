# Isolate — Architecture

## Overview

Isolate is a four-layer system: blockchain (Soroban contracts), backend (Node.js API + indexer), frontend (Next.js), and mobile (Expo). All financial logic lives on-chain; the backend provides indexing, off-chain metadata, and API aggregation.

```
┌─────────────────────────────────────────────────────────────┐
│                        USER LAYER                           │
│   Buyer · Seller · Driver · Mediator · Admin                │
│   Freighter / Albedo / Lobstr / WalletConnect wallets       │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTPS / WebSocket
┌──────────────────────────▼──────────────────────────────────┐
│                     FRONTEND LAYER                          │
│   Next.js 14 App Router · Stellar SDK · Supabase client     │
└──────────┬────────────────────────────────────┬─────────────┘
           │ REST / WebSocket                    │ Direct RPC
┌──────────▼────────────────────────────────────▼─────────────┐
│                     BACKEND LAYER                           │
│   Express REST API · OpenAPI · Soroban Event Indexer        │
│   Supabase (metadata) · Pinata (IPFS PoD storage)          │
└──────────┬────────────────────────────────────┬─────────────┘
           │ Soroban RPC / Horizon               │ Supabase / Redis
┌──────────▼──────────────────┐   ┌─────────────▼────────────┐
│      BLOCKCHAIN LAYER       │   │       DATA LAYER         │
│   Stellar Network           │   │   Supabase (PostgreSQL)  │
│   Soroban Smart Contracts   │   │   Redis (cache/sessions) │
│   Horizon API               │   │   Pinata IPFS (PoD)      │
└─────────────────────────────┘   └──────────────────────────┘
```

---

## Blockchain Layer — Soroban Contracts

### Contract Modules

```
contracts/amana_escrow/src/
├── lib.rs          # Entry point — routes invocations
├── escrow.rs       # Trade lifecycle: create, confirm, cancel
├── delivery.rs     # Proof-of-Delivery: submit, verify, reject
├── dispute.rs      # Mediator quorum: raise, vote, resolve
├── settlement.rs   # Fund release, refund, platform fee
├── roles.rs        # Role validation (buyer, seller, driver, mediator)
├── storage.rs      # Storage key definitions
├── events.rs       # Event emission helpers
├── errors.rs       # Error enum
└── types.rs        # Trade, Delivery, Dispute, Settlement types
```

### Trade State Machine

```
CREATED → CONFIRMED → IN_TRANSIT → DELIVERED → SETTLED
    │           │           │           │
    │           │           │           └── DISPUTED → RESOLVED
    │           │           └── DELIVERY_REJECTED → DISPUTED
    └── CANCELLED           └── DEADLINE_EXPIRED → REFUNDED
```

### Storage Model

| Key | Type | Description |
|-----|------|-------------|
| `TRADE:{id}` | `Trade` | Full trade record |
| `DELIVERY:{trade_id}` | `DeliveryProof` | PoD submission |
| `DISPUTE:{trade_id}` | `Dispute` | Dispute record + votes |
| `MEDIATOR_VOTES:{trade_id}` | `Vec<(Address, Verdict)>` | Mediator votes |
| `PLATFORM_FEE_VAULT` | `i128` | Accumulated platform fees |
| `ADMIN` | `Address` | Protocol admin |

### Event Schema

| Event | Topics | Data |
|-------|--------|------|
| `trade_created` | `[trade_id, buyer, seller]` | `amount, asset, deadline` |
| `trade_confirmed` | `[trade_id, seller]` | `timestamp` |
| `delivery_submitted` | `[trade_id, driver]` | `ipfs_cid` |
| `delivery_verified` | `[trade_id, buyer]` | `timestamp` |
| `delivery_rejected` | `[trade_id, buyer]` | `reason` |
| `dispute_raised` | `[trade_id, caller]` | `evidence_cid` |
| `mediator_voted` | `[trade_id, mediator]` | `verdict` |
| `dispute_resolved` | `[trade_id]` | `outcome, release_amount` |
| `funds_released` | `[trade_id, seller]` | `amount, fee` |
| `buyer_refunded` | `[trade_id, buyer]` | `amount` |
| `trade_cancelled` | `[trade_id]` | `reason` |

### Asset Handling

All fund custody is held by the contract account using the Stellar token interface (SEP-41). Supported assets: XLM (native), cNGN, USDC, EURC, and any SEP-41 token. Asset address is stored per-trade at creation.

---

## Backend Layer

### Service Map

```
backend/src/
├── server.ts             # Express bootstrap
├── indexer/
│   ├── EventPoller.ts    # Polls Horizon for Soroban events
│   ├── EventProcessor.ts # Parses XDR events
│   └── handlers/         # One handler per event type
├── api/
│   ├── routes/           # REST handlers
│   │   ├── trades.ts
│   │   ├── delivery.ts
│   │   ├── disputes.ts
│   │   └── settlement.ts
│   └── middleware/       # Auth, rate limiting, validation
├── docs/
│   └── openapi.yaml      # OpenAPI 3.0 spec (source of truth)
├── services/
│   ├── StellarService.ts # Horizon + Soroban RPC wrapper
│   ├── PinataService.ts  # IPFS upload/retrieval
│   └── SupabaseService.ts
└── config/               # Env config with Zod validation
```

### Authentication

SEP-10 Stellar Web Authentication — wallet keypair is the identity. Short-lived JWTs issued after challenge signing.

### Indexer Design

```
Horizon /events stream (cursor-based polling)
        │
        ▼
EventProcessor (parse XDR topics + data)
        │
        ├── TradeHandler       → updates trades table
        ├── DeliveryHandler    → updates deliveries table
        ├── DisputeHandler     → updates disputes + votes tables
        └── SettlementHandler  → updates settlements table
              │
              ▼
        Supabase + Redis pub/sub → WebSocket → Frontend
```

---

## Frontend Layer

### Page Structure

```
frontend/src/app/
├── page.tsx              # Landing
├── dashboard/            # Trade overview
├── trade/
│   ├── create/           # Create trade wizard
│   ├── [id]/             # Trade detail
│   └── [id]/deliver/     # PoD submission
├── dispute/[id]/         # Dispute management
└── profile/              # Trade history, reputation
```

### Wallet Integration

Freighter and Albedo via `@stellar/freighter-api`. WalletConnect via Stellar bridge. SEP-10 auth flow built into the `useAuth` hook.

---

## Mobile Layer

Expo React Native sharing business logic with frontend via `packages/core`. Push notifications via Expo Notifications for trade status updates. Offline-first with MMKV persistence and queued transaction signing.

---

## Data Layer

### Supabase Schema (core tables)

```sql
trades (id, buyer, seller, amount, asset, status, deadline, loss_ratio, created_at)
deliveries (trade_id, driver, ipfs_cid, submitted_at, verified_at)
disputes (trade_id, raised_by, evidence_cid, status, resolved_at)
mediator_votes (trade_id, mediator, verdict, cast_at)
settlements (trade_id, seller_amount, fee_amount, refund_amount, settled_at)
users (address, display_name, reputation_score, trade_count)
```

### Redis Usage

| Key | TTL | Purpose |
|-----|-----|---------|
| `session:{jwt_id}` | 1h | JWT revocation |
| `rate:{ip}` | 1min | Rate limiting |
| `trade_cache:{id}` | 30s | Frequently-read trade state |
| `pubsub:trade:{id}` | — | Real-time WS push |

---

## Security Architecture

| Concern | Mitigation |
|---------|-----------|
| Fund custody | Soroban contract — no EOA holds funds |
| Auth | SEP-10 wallet challenge; short-lived JWTs |
| Role enforcement | All contract functions validate caller role on-chain |
| Re-entrancy | Soroban synchronous execution model |
| Integer overflow | Rust `checked_add`/`checked_mul` throughout |
| Input validation | Zod on backend; contract validates all params |
| Secret scanning | Gitleaks on every PR |
| Dependency audit | cargo-audit + npm audit in CI |

# Contributing to Isolate

Thank you for your interest in contributing to Isolate — a decentralized escrow protocol for agricultural trade on Stellar. Contributions of all kinds are welcome: code, documentation, translations, design, and testing.

## Table of Contents

1. [Code of Conduct](#1-code-of-conduct)
2. [Ways to Contribute](#2-ways-to-contribute)
3. [Development Setup](#3-development-setup)
4. [Branching & Commit Convention](#4-branching--commit-convention)
5. [Opening a Pull Request](#5-opening-a-pull-request)
6. [Issue Labels](#6-issue-labels)
7. [Smart Contract Contributions](#7-smart-contract-contributions)
8. [Frontend Contributions](#8-frontend-contributions)
9. [Backend Contributions](#9-backend-contributions)
10. [Testing Requirements](#10-testing-requirements)
11. [Wave-Ready Funded Issues](#11-wave-ready-funded-issues)
12. [Getting Help](#12-getting-help)

---

## 1. Code of Conduct

This project follows the [Contributor Covenant](CODE_OF_CONDUCT.md). Be respectful and constructive.

---

## 2. Ways to Contribute

- **Code**: bug fixes, new features, performance improvements, security hardening
- **Documentation**: fix typos, expand thin sections, add missing guides
- **Testing**: improve coverage, add edge case tests, write E2E scenarios
- **Translations**: translate UI strings and docs into local languages
- **Design**: UI/UX improvements, accessibility fixes, icons and illustrations

---

## 3. Development Setup

### Prerequisites

| Tool | Version |
|------|---------|
| Rust | 1.75+ |
| Node.js | 20+ |
| pnpm | 10+ |
| Stellar CLI | latest |
| Docker | 24+ |

### Install

```bash
git clone https://github.com/ikufidelade1-cpu/isolate.git
cd isolate
pnpm install
cp .env.staging.example .env
```

### Contracts

```bash
cd contracts/amana_escrow
cargo build
cargo test
```

### Backend

```bash
cd backend
cp .env.example .env
pnpm dev
```

### Frontend

```bash
cd frontend
cp .env.example .env.local
pnpm dev
```

---

## 4. Branching & Commit Convention

### Branch Names

```
feat/short-description
fix/short-description
docs/short-description
test/short-description
chore/short-description
```

### Commit Messages (Conventional Commits)

```
feat(contracts): add mediator quorum dispute resolution
fix(backend): correct settlement event handler race condition
docs(user-guide): add dispute resolution section
test(contracts): add edge cases for deadline expiry refund
chore(deps): bump stellar-sdk to 13.0.0
```

---

## 5. Opening a Pull Request

1. Fork and create your branch from `main`
2. Make changes with tests
3. Run `pnpm test` and `pnpm lint`
4. Open PR against `main` — fill in the template
5. Link the relevant issue (`Closes #123`)
6. Wait for CI and maintainer review

PRs touching `contracts/` require 2 maintainer approvals.

---

## 6. Issue Labels

| Label | Description |
|-------|-------------|
| `contracts` | Soroban smart contract work |
| `frontend` | Next.js frontend |
| `backend` | Node.js API and indexer |
| `mobile` | Expo React Native app |
| `docs` | Documentation only |
| `testing` | Tests and coverage |
| `good-first-issue` | Great for first-time contributors |
| `wave-ready` | Funded Drips Wave issue |
| `bug` | Something is broken |
| `security` | Security related |
| `performance` | Performance improvement |

---

## 7. Smart Contract Contributions

- All public functions must have doc comments
- Use `checked_add`/`checked_mul` — no unchecked arithmetic
- All state mutations must emit a corresponding event
- New error variants go in `errors.rs`
- New shared types go in `types.rs`

```bash
cargo test
cargo clippy --all-targets -- -D warnings
cargo fmt --all -- --check
```

---

## 8. Frontend Contributions

- Use the existing component structure — pages in `app/`, reusable UI in `components/`
- All data fetching via React Query / SWR — no raw fetch in components
- Follow the existing Tailwind CSS patterns — no inline styles
- All interactive elements must be keyboard navigable with visible focus rings

---

## 9. Backend Contributions

- New routes in `backend/src/api/routes/` following existing patterns
- All database changes need a migration in `backend/src/db/migrations/`
- Event handlers must be idempotent
- All endpoints need integration tests

---

## 10. Testing Requirements

| Workspace | Minimum Coverage | Tool |
|-----------|-----------------|------|
| contracts | 85% lines | cargo-tarpaulin |
| frontend | 80% lines | vitest (v8) |
| backend | 60% lines | jest |

---

## 11. Wave-Ready Funded Issues

Issues labelled `wave-ready` carry funding through the Drips Wave program.

| Difficulty | Points |
|-----------|--------|
| `trivial` | 100 |
| `medium` | 150 |
| `high` | 200 |

Comment on the issue to claim it before starting work.

---

## 12. Getting Help

- **GitHub Discussions** — questions and ideas
- **GitHub Issues** — bugs and feature requests
- Search existing issues before opening a new one

*Building trust infrastructure for agricultural trade — together.*

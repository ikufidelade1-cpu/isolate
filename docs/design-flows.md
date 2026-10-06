# Isolate — Design Flows

## Table of Contents

1. [Wallet Connection & Onboarding](#1-wallet-connection--onboarding)
2. [Creating a Trade](#2-creating-a-trade)
3. [Seller Confirmation](#3-seller-confirmation)
4. [Proof-of-Delivery Submission](#4-proof-of-delivery-submission)
5. [Buyer Verification & Fund Release](#5-buyer-verification--fund-release)
6. [Dispute Flow](#6-dispute-flow)
7. [Deadline Expiry & Refund](#7-deadline-expiry--refund)
8. [Cancellation Flow](#8-cancellation-flow)
9. [Error & Edge Case Flows](#9-error--edge-case-flows)

---

## 1. Wallet Connection & Onboarding

```
[Landing Page]
    │
    ▼
[Connect Wallet Modal]
    ├── Freighter (recommended)
    ├── Albedo
    ├── Lobstr
    └── WalletConnect
    │
    ▼
[Wallet returns public key G...]
    │
    ▼
[Backend: GET /auth/challenge?account=G...]
    │
    ▼
[Wallet signs challenge transaction]
    │
    ▼
[Backend: POST /auth/token → JWT issued]
    │
    ▼
[Dashboard: My Trades, Create Trade, Profile]
```

---

## 2. Creating a Trade

**Actor**: Buyer

```
[Dashboard → Create Trade]
    │
    ▼
[Step 1: Trade Details]
    ├── Seller address (G...)
    ├── Commodity description
    ├── Trade amount + asset (XLM / cNGN / USDC)
    ├── Delivery deadline (date picker)
    └── Loss sharing ratio (buyer/seller split on damage)
    │
    ▼
[Step 2: Driver Details (optional at creation)]
    └── Driver address (can be added by seller later)
    │
    ▼
[Step 3: Review & Confirm]
    ├── Summary of all terms
    ├── Estimated gas fee
    └── "Lock Funds & Create Trade" button
    │
    ▼
[Wallet signs create_trade() transaction]
    │
    ▼
[Funds locked in Soroban contract vault]
    │
    ▼
[Seller notified — trade awaiting confirmation]
    │
    ▼
[Trade Detail page: status CREATED]
```

---

## 3. Seller Confirmation

**Actor**: Seller

```
[Seller receives notification: "New trade awaiting your confirmation"]
    │
    ▼
[Trade Detail page — Seller view]
    ├── Buyer address, amount, commodity, deadline
    ├── Loss sharing ratio
    └── "Confirm Trade" button
    │
    ▼
[Seller optionally assigns driver address]
    │
    ▼
[Wallet signs confirm_trade() transaction]
    │
    ▼
[Trade status → CONFIRMED]
    │
    ▼
[Driver notified if assigned]
    │
    ▼
[Goods dispatched off-chain]
```

---

## 4. Proof-of-Delivery Submission

**Actor**: Driver (or Seller)

```
[Driver arrives at delivery point]
    │
    ▼
[Mobile app → Trade → Submit Delivery]
    │
    ▼
[Step 1: Record or upload delivery video]
    ├── Mobile camera capture (recommended)
    └── Upload existing file
    │
    ▼
[Step 2: Video uploaded to IPFS via Pinata]
    │
    ▼
[Step 3: IPFS CID returned]
    │
    ▼
[Wallet signs submit_delivery(trade_id, driver, ipfs_cid)]
    │
    ▼
[delivery_submitted event emitted]
    │
    ▼
[Trade status → IN_TRANSIT → DELIVERED (pending buyer verification)]
    │
    ▼
[Buyer notified: "Delivery submitted — please verify"]
```

---

## 5. Buyer Verification & Fund Release

**Actor**: Buyer

```
[Buyer receives notification: "Delivery submitted — review required"]
    │
    ▼
[Trade Detail page — Buyer view]
    ├── IPFS video player (fetched from Pinata)
    ├── Driver address, submission timestamp
    ├── "Verify Delivery" button
    └── "Reject Delivery" button (requires reason)
    │
    ▼ (Buyer verifies)
[Wallet signs verify_delivery(trade_id, buyer)]
    │
    ▼
[Contract auto-executes release_funds()]
    ├── Seller receives: amount - 1% platform fee
    ├── Platform fee held in contract vault
    └── delivery_verified + funds_released events emitted
    │
    ▼
[Seller notified: "Payment received"]
[Trade status → SETTLED]
```

---

## 6. Dispute Flow

**Actor**: Buyer (rejects delivery)

```
[Buyer clicks "Reject Delivery"]
    │
    ▼
[Rejection Modal]
    ├── Reason text input
    ├── Upload counter-evidence (optional, stored on IPFS)
    └── "Submit Rejection" button
    │
    ▼
[Wallet signs reject_delivery(trade_id, buyer, reason)]
    │
    ▼
[Dispute auto-raised → dispute_raised event]
    │
    ▼
[Mediator quorum notified]
    ├── Up to 3 mediators assigned (protocol-level or community)
    └── 48h voting window opens
    │
    ▼
[Mediators review evidence]
    ├── Watch PoD video on IPFS
    ├── Review buyer's counter-evidence
    └── Cast vote: FOR_SELLER / FOR_BUYER / SPLIT
    │
    ▼
[Wallet signs submit_mediator_vote(trade_id, mediator, verdict)]
    │
    ▼ (quorum reached — 2 of 3 votes)
[resolve_dispute() executed]
    │
    ├── FOR_SELLER: funds released to seller (minus fee + loss ratio)
    ├── FOR_BUYER: full refund to buyer
    └── SPLIT: loss ratio applied — each party gets their share
    │
    ▼
[dispute_resolved event → both parties notified]
```

---

## 7. Deadline Expiry & Refund

```
[Delivery deadline passes — no delivery submitted]
    │
    ▼
[Contract detects DEADLINE_EXPIRED state on next interaction]
    │
    ▼
[refund_buyer() callable by buyer OR auto-triggered by backend]
    │
    ▼
[Full amount returned to buyer's wallet]
    │
    ▼
[buyer_refunded event → buyer notified]
[Trade status → REFUNDED]
```

---

## 8. Cancellation Flow

**Before seller confirms:**
```
[Buyer → Trade Detail → Cancel Trade]
    │
    ▼
[Wallet signs cancel_trade(trade_id, buyer)]
    │
    ▼
[Full refund to buyer — trade_cancelled event]
```

**After seller confirms (requires mutual consent):**
```
[Either party requests cancellation → governance proposal]
    │
    ▼
[Other party approves on-chain]
    │
    ▼
[Funds returned per agreed split]
```

---

## 9. Error & Edge Case Flows

### Transaction Rejected by Wallet
```
User rejects in wallet
    → Toast: "Transaction cancelled. No funds moved."
    → Trade state unchanged
```

### Insufficient Balance
```
Frontend pre-validates balance before building XDR
    → Error modal: "Insufficient balance. Need X XLM + ~0.01 XLM for fees."
    → CTA: "Top up with XLM"
```

### IPFS Upload Failure
```
Pinata upload fails during PoD submission
    → Retry up to 3 times
    → If still failing: "Upload failed. Please try again or use a smaller file."
    → Trade state not updated until CID confirmed
```

### Mediator Timeout
```
Voting window expires without quorum
    → Dispute escalated to protocol admin
    → Admin has 24h to cast deciding vote
    → If admin fails: refund buyer by default
```

### Duplicate Verification Attempt
```
Buyer calls verify_delivery() twice
    → Contract checks: already SETTLED
    → Returns AlreadySettled error
    → Frontend: "This trade has already been completed."
```

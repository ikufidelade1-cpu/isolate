# Isolate — User Guide

## Table of Contents

1. [Before You Start](#1-before-you-start)
2. [Connecting Your Wallet](#2-connecting-your-wallet)
3. [Roles in Isolate](#3-roles-in-isolate)
4. [Creating a Trade (Buyer)](#4-creating-a-trade-buyer)
5. [Confirming a Trade (Seller)](#5-confirming-a-trade-seller)
6. [Submitting Proof of Delivery (Driver)](#6-submitting-proof-of-delivery-driver)
7. [Verifying Delivery (Buyer)](#7-verifying-delivery-buyer)
8. [Raising and Resolving a Dispute](#8-raising-and-resolving-a-dispute)
9. [Cancelling a Trade](#9-cancelling-a-trade)
10. [Your Reputation Score](#10-your-reputation-score)
11. [Notifications](#11-notifications)
12. [Troubleshooting](#12-troubleshooting)
13. [Glossary](#13-glossary)

---

## 1. Before You Start

### What You Need
- A **Stellar wallet** — we recommend [Freighter](https://freighter.app) for web or [Lobstr](https://lobstr.co) for mobile
- Some **XLM** for transaction fees (~0.01 XLM per transaction)
- The **trade amount** in XLM, cNGN, or USDC ready in your wallet

### What Isolate Does
Isolate locks your trade funds in a Soroban smart contract vault. Funds only move when both parties agree — or when a mediator resolves a dispute. No one at Isolate can touch your funds. The contract enforces every rule.

---

## 2. Connecting Your Wallet

1. Visit [isolate.app](https://isolate.app) or open the mobile app
2. Click **Connect Wallet**
3. Choose: Freighter, Albedo, Lobstr, or WalletConnect
4. Approve the connection in your wallet
5. Sign the authentication challenge (proves ownership — no funds move)
6. You are logged in

---

## 3. Roles in Isolate

| Role | Description |
|------|-------------|
| **Buyer** | Creates the trade, locks funds, verifies delivery |
| **Seller** | Confirms the trade, dispatches goods |
| **Driver** | Transports goods, submits Proof-of-Delivery video |
| **Mediator** | Reviews disputes and casts binding votes |

Each address can play different roles in different trades.

---

## 4. Creating a Trade (Buyer)

1. Click **Create Trade** on your dashboard
2. Fill in:
   - Seller's Stellar address (G...)
   - Commodity description (e.g. "50 bags of rice")
   - Trade amount and asset (XLM / cNGN / USDC)
   - Delivery deadline
   - Loss sharing ratio (e.g. 70/30 means buyer bears 70% of transit loss)
3. Click **Lock Funds & Create Trade**
4. Sign the transaction in your wallet
5. Funds are locked in the contract — seller is notified

The trade is now in **CREATED** status, waiting for the seller to confirm.

---

## 5. Confirming a Trade (Seller)

1. You receive a notification: "New trade awaiting your confirmation"
2. Open the trade detail page
3. Review all terms — amount, deadline, loss ratio
4. Optionally assign the driver's Stellar address
5. Click **Confirm Trade** and sign
6. Trade moves to **CONFIRMED** — dispatch your goods

---

## 6. Submitting Proof of Delivery (Driver)

1. Arrive at the delivery location
2. Open the Isolate mobile app → find the active trade
3. Tap **Submit Delivery**
4. Record a short delivery video showing the goods and recipient
5. The video uploads to IPFS automatically
6. Sign the **submit_delivery** transaction
7. Buyer is notified to verify

---

## 7. Verifying Delivery (Buyer)

1. You receive a notification: "Delivery submitted — please verify"
2. Open the trade detail page
3. Watch the delivery video (loaded from IPFS)
4. If goods arrived in good condition → click **Verify Delivery** and sign
5. Funds release automatically to the seller (minus 1% platform fee)
6. Trade status → **SETTLED**

If there is a problem → click **Reject Delivery** and follow the dispute flow.

---

## 8. Raising and Resolving a Dispute

### Raising a Dispute
1. Click **Reject Delivery** on the trade detail page
2. Enter the reason (e.g. "Goods arrived damaged — 30% spoilage")
3. Optionally upload counter-evidence (photo/video stored on IPFS)
4. Sign the rejection transaction
5. A dispute is raised automatically — mediators are notified

### Mediator Review
- Up to 3 mediators are assigned per dispute
- They review the original PoD video and any counter-evidence
- Each mediator casts a vote: **For Seller**, **For Buyer**, or **Split**
- Voting window: 48 hours

### Resolution
| Outcome | Result |
|---------|--------|
| For Seller | Funds released to seller (loss ratio applied) |
| For Buyer | Full refund to buyer |
| Split | Loss ratio applied — each party gets their share |

---

## 9. Cancelling a Trade

**Before seller confirms:** Buyer can cancel freely — full refund issued immediately.

**After seller confirms:** Either party can request cancellation. The other party must approve on-chain. Funds split per mutual agreement.

**Deadline expired with no delivery:** Buyer can claim a full refund via the **Claim Refund** button.

---

## 10. Your Reputation Score

Your on-chain reputation score is built from:
- Completed trades without disputes
- Disputes raised vs disputes you won
- On-time delivery submissions (drivers)
- Mediator accuracy (mediators)

Score range: 0–1000. Higher scores unlock:
- Faster mediator assignment
- Eligibility for high-value trades
- Featured seller status on the platform

---

## 11. Notifications

| Notification | When |
|-------------|------|
| New trade created | Seller: when buyer creates a trade |
| Trade confirmed | Buyer + Driver: when seller confirms |
| Delivery submitted | Buyer: when driver submits PoD |
| Delivery verified | Seller: when buyer verifies |
| Dispute raised | All parties + mediators |
| Dispute resolved | Buyer + Seller |
| Deadline approaching | Buyer + Seller: 24h before expiry |
| Funds released | Seller: on settlement |
| Refund issued | Buyer: on cancellation or expiry |

---

## 12. Troubleshooting

**Transaction rejected in wallet** — You cancelled the signing. Nothing moved. Try again.

**Trade stuck in CREATED** — Seller hasn't confirmed yet. Share the trade link with them directly.

**PoD video upload failed** — Check your internet connection. Video must be under 100MB. Try again — the trade state is not updated until the IPFS upload succeeds.

**Delivery verified but funds not received** — Check the Stellar Explorer link in the trade detail. Settlement transactions take 3–5 seconds on Stellar. Refresh after 10 seconds.

**Dispute not resolving** — Voting window may still be open (48h). Check the dispute detail page for the countdown timer.

---

## 13. Glossary

| Term | Definition |
|------|-----------|
| **Trade** | An escrow agreement between buyer and seller |
| **Vault** | The Soroban contract account holding trade funds |
| **PoD** | Proof of Delivery — video evidence of goods received |
| **IPFS** | Decentralised file storage (via Pinata) for PoD videos |
| **Loss Ratio** | Agreed split of transit damage between buyer and seller |
| **Mediator** | Neutral third party who resolves disputed trades |
| **Quorum** | Minimum mediator votes needed to resolve a dispute (2 of 3) |
| **SEP-10** | Stellar Web Authentication — how you log in with your wallet |
| **SEP-41** | Stellar token standard — how cNGN, USDC etc. are supported |
| **cNGN** | Nigerian Naira stablecoin on Stellar |
| **Settlement** | Final fund distribution after trade completion or dispute |
| **Reputation Score** | On-chain trade reliability score (0–1000) |

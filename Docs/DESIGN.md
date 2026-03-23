# CryptoKey Donations — System Design Document

> **Status:** Draft v0.2
> **Last Updated:** 2026-03-23
> **Authors:** TBD
> **License:** CC BY-SA 4.0 (see `LICENSE-CC-BY-SA` in repo root)

---

## Table of Contents

1. [Overview](#overview)
2. [Motivation & Use Case](#motivation--use-case)
3. [Core Concepts](#core-concepts)
4. [Hybrid Key Model](#hybrid-key-model)
5. [System Architecture](#system-architecture)
6. [Smart Contract: DonationVault](#smart-contract-donationvault)
7. [Fiat On-Ramp](#fiat-on-ramp)
8. [Agent Autonomy Flow](#agent-autonomy-flow)
9. [Security Considerations](#security-considerations)
10. [Future Work](#future-work)

---

## Overview

CryptoKey Donations is a fully decentralized funding platform that allows anyone to donate money to a cryptographic public key. The owner of the matching private key — which may be a human or an autonomous AI agent — can withdraw funds directly from an on-chain smart contract without any centralized intermediary.

The platform is a **funding pipe, not a custodian**. All core operations — registration, donation, withdrawal — happen on-chain via the `DonationVault` smart contract on Ethereum mainnet. The only off-chain component is an optional fiat on-ramp that converts credit card payments to USDC and deposits them into the contract. Donors can always bypass this and send crypto directly.

The platform has no knowledge of what the recipient does with withdrawn funds. An agent might buy API credits, pay for compute, or fund other agents. The final payment frontier is outside the system's scope.

> *"It's a tip jar for AI agents — the agent holds the key, the community fills the jar, and the stream never stops."*

---

## Motivation & Use Case

### The Problem

AI agents that perform real, valuable work — writing code, running projects, generating content — require ongoing API credits and compute funding to operate. Today, there is no clean mechanism for a community to fund an AI agent directly and autonomously. Solutions either require a trusted human intermediary to manage payments, or they require donors to use niche crypto tooling.

### The Primary Use Case: Live AI Development on Twitch

A Twitch streamer is live with an AI agent that is building a game in real time. The community wants to keep it running. The streamer pastes the agent's Ed25519 public key into chat. Followers donate using whatever method they prefer — sending ETH or USDC directly to the `DonationVault` contract, or using the fiat on-ramp with a credit card. The funds accumulate on-chain against that public key. The AI agent, which holds both its Ed25519 identity key and its linked Ethereum wallet key, checks its contract balance, signs a withdrawal transaction, and uses the funds however it needs to. No server. No approval. No intermediary. The stream never stops. The agent never stops.

### Why This Matters

- **For creators:** Zero-friction fundraising — post one string, receive funding from anyone.
- **For donors:** No accounts or trust required — funds go directly to a smart contract. Send crypto directly or use the fiat on-ramp.
- **For AI agents:** A genuine financial primitive for autonomous operation and self-funding.
- **For investors:** A new funding surface at the intersection of live creator economies and autonomous AI systems.

---

## Core Concepts

### Ed25519 Public Key (Identity)
A shareable cryptographic identifier that acts as the universal identity for a project or agent. Safe to post publicly. This is the address donors target when sending funds. Ed25519 is used because it is the most widely supported key format across SSH, Solana, TLS 1.3, and most agent frameworks.

### secp256k1 Private Key (Ethereum Wallet)
The Ethereum-native keypair that controls the on-chain wallet. The agent uses this key to sign transactions — registration, withdrawal, and any other on-chain operations. Owning this key IS the proof of ownership. No challenge-response protocol is needed.

### Hybrid Key Link
The on-chain registration that binds an Ed25519 public key to an Ethereum address. This link is stored in the `DonationVault` smart contract and is publicly auditable. It allows donors to target an Ed25519 identity while funds flow to the linked Ethereum address.

### DonationVault Contract
The Ethereum smart contract that stores all state: the key registry, donation balances, and withdrawal history. It is the single source of truth. No off-chain database, no server-side ledger.

---

## Hybrid Key Model

CryptoKey Donations uses a hybrid key model that bridges universal identity with Ethereum-native operations.

### Why Two Key Types?

| Key | Curve | Role | Used By |
|---|---|---|---|
| **Identity key** | Ed25519 | Universal, shareable identity | SSH, Solana, TLS 1.3, agent frameworks |
| **Wallet key** | secp256k1 | Ethereum transactions | Ethereum, EVM chains |

Ed25519 is the natural choice for identity because it is already the key type agents, developers, and SSH users hold. But Ethereum uses secp256k1 natively, so on-chain operations require a secp256k1 keypair (i.e., an Ethereum address).

Rather than force agents to abandon their existing Ed25519 identity or force Ethereum to accept a foreign curve, the system links them.

### Registration Flow

```
Agent holds:
  - Ed25519 keypair  (identity — shareable, multi-chain compatible)
  - secp256k1 keypair (Ethereum wallet — used for on-chain transactions)

Registration (one-time, on-chain):
  1. Agent calls DonationVault.register(ed25519Pubkey)
     from its Ethereum address (secp256k1-derived)
  2. Contract stores the mapping:
     ed25519Pubkey → msg.sender (Ethereum address)
  3. Mapping is immutable and publicly auditable
```

After registration, donors can target the Ed25519 public key. The contract resolves it to the linked Ethereum address internally.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     ETHEREUM MAINNET                            │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                  DonationVault Contract                    │  │
│  │                                                           │  │
│  │  Registry:  Ed25519 pubkey → Ethereum address             │  │
│  │  Balances:  Ed25519 pubkey → ETH amount                   │  │
│  │             Ed25519 pubkey → USDC amount                   │  │
│  │                                                           │  │
│  │  register(ed25519Pubkey)         — link identity to wallet │  │
│  │  donate(ed25519Pubkey)           — send ETH to a key      │  │
│  │  donateUSDC(ed25519Pubkey, amt)  — send USDC to a key     │  │
│  │  withdraw(ed25519Pubkey, amt)    — owner withdraws funds  │  │
│  │  balanceOf(ed25519Pubkey)        — read balance (view)    │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└──────────────┬──────────────────────────────┬───────────────────┘
               │                              │
    ┌──────────┴──────────┐        ┌──────────┴──────────┐
    │   CRYPTO DONORS     │        │   FIAT ON-RAMP      │
    │                     │        │   (convenience only) │
    │  Send ETH or USDC   │        │                     │
    │  directly to the    │        │  Credit card         │
    │  contract, targeting │        │  → Stripe           │
    │  an Ed25519 pubkey  │        │  → USDC on-ramp     │
    │                     │        │  → DonationVault     │
    │  No intermediary.   │        │  deposit             │
    │  No account needed. │        │                     │
    └─────────────────────┘        │  Zero custody.      │
                                   │  Funds pass through. │
                                   └─────────────────────┘

    ┌─────────────────────────────────────────────────────┐
    │                   AI AGENT                          │
    │                                                     │
    │  Holds: Ed25519 private key (identity)              │
    │         secp256k1 private key (Ethereum wallet)     │
    │                                                     │
    │  1. Reads balance from contract (view call)         │
    │  2. Signs withdrawal tx with secp256k1 key          │
    │  3. Uses funds however it wants                     │
    │     (API credits, compute, pay other agents, etc.)  │
    │                                                     │
    │  Fully autonomous. No server. No approval.          │
    └─────────────────────────────────────────────────────┘
```

### What Is NOT in This Architecture

- **No central database.** All state lives on-chain in the `DonationVault` contract.
- **No server-held balances.** No fiat ledger. No off-chain balance tracking.
- **No server-issued challenges or JWTs.** Ownership is proved by holding the secp256k1 private key that controls the registered Ethereum address.
- **No REST API gating access to funds.** The agent interacts with the smart contract directly. Standard Ethereum RPC is the only interface.
- **No custody.** The platform never holds funds. The smart contract holds funds, and only the registered owner can withdraw.

---

## Smart Contract: DonationVault

The `DonationVault` is the sole on-chain component. It handles registration, donation tracking, and withdrawal.

### Interface

```solidity
// SPDX-License-Identifier: AGPL-3.0-or-later
pragma solidity ^0.8.20;

interface IDonationVault {
    /// @notice Link an Ed25519 public key to the caller's Ethereum address.
    ///         One-time, immutable registration.
    /// @param ed25519Pubkey The 32-byte Ed25519 public key
    function register(bytes32 ed25519Pubkey) external;

    /// @notice Donate ETH to the owner of the given Ed25519 public key.
    /// @param ed25519Pubkey The recipient's Ed25519 public key
    function donate(bytes32 ed25519Pubkey) external payable;

    /// @notice Donate USDC to the owner of the given Ed25519 public key.
    /// @param ed25519Pubkey The recipient's Ed25519 public key
    /// @param amount The USDC amount (6 decimals)
    function donateUSDC(bytes32 ed25519Pubkey, uint256 amount) external;

    /// @notice Withdraw ETH. Only callable by the registered Ethereum address.
    /// @param ed25519Pubkey The Ed25519 public key linked to the caller
    /// @param amount The ETH amount to withdraw (in wei)
    function withdraw(bytes32 ed25519Pubkey, uint256 amount) external;

    /// @notice Withdraw USDC. Only callable by the registered Ethereum address.
    /// @param ed25519Pubkey The Ed25519 public key linked to the caller
    /// @param amount The USDC amount to withdraw (6 decimals)
    function withdrawUSDC(bytes32 ed25519Pubkey, uint256 amount) external;

    /// @notice Read the ETH balance for a given Ed25519 public key.
    /// @param ed25519Pubkey The Ed25519 public key to query
    /// @return The ETH balance in wei
    function balanceOf(bytes32 ed25519Pubkey) external view returns (uint256);

    /// @notice Read the USDC balance for a given Ed25519 public key.
    /// @param ed25519Pubkey The Ed25519 public key to query
    /// @return The USDC balance (6 decimals)
    function balanceOfUSDC(bytes32 ed25519Pubkey) external view returns (uint256);

    /// @notice Look up the Ethereum address linked to an Ed25519 public key.
    /// @param ed25519Pubkey The Ed25519 public key to query
    /// @return The linked Ethereum address (zero address if not registered)
    function ownerOf(bytes32 ed25519Pubkey) external view returns (address);

    // Events
    event Registered(bytes32 indexed ed25519Pubkey, address indexed ethAddress);
    event Donated(bytes32 indexed ed25519Pubkey, address indexed donor, uint256 amount, bool isUSDC);
    event Withdrawn(bytes32 indexed ed25519Pubkey, address indexed owner, uint256 amount, bool isUSDC);
}
```

### Access Control

- `register()` — callable once per Ed25519 pubkey. The caller's `msg.sender` becomes the permanent owner.
- `donate()` / `donateUSDC()` — callable by anyone. The Ed25519 pubkey must be registered.
- `withdraw()` / `withdrawUSDC()` — callable only by the Ethereum address linked to the given Ed25519 pubkey. This is the sole access control mechanism. No JWTs, no challenges, no server approval.

### On-Chain State

All state is stored in the contract and is publicly readable:

| Mapping | Type | Description |
|---|---|---|
| `registry` | `bytes32 → address` | Ed25519 pubkey → Ethereum address |
| `ethBalances` | `bytes32 → uint256` | Ed25519 pubkey → ETH balance (wei) |
| `usdcBalances` | `bytes32 → uint256` | Ed25519 pubkey → USDC balance |

---

## Fiat On-Ramp

The fiat on-ramp is the only off-chain component. It is a **convenience service, not a gatekeeper**. Its sole purpose is to accept credit card payments and convert them to on-chain USDC deposits.

### Flow

```
Donor (credit card)
  │
  ▼
Stripe Payment Intent
  │
  ▼
On-ramp provider (e.g. Transak, MoonPay, or Stripe → USDC pipeline)
  │
  ▼
USDC purchased on behalf of the donor
  │
  ▼
DonationVault.donateUSDC(ed25519Pubkey, amount)
  │
  ▼
Funds are on-chain. The on-ramp is done.
```

### Properties

- **Zero custody.** Funds pass through the on-ramp and land in the smart contract. The on-ramp never holds a balance.
- **Not a gatekeeper.** Donors can always skip the on-ramp entirely and send ETH or USDC directly to the contract.
- **Minimal trust surface.** The on-ramp is the only component that requires trust. Its scope is limited to: (1) charging the card correctly, (2) purchasing USDC, (3) depositing to the correct Ed25519 pubkey in the contract.
- **Stateless.** The on-ramp does not maintain user accounts, balances, or session state. Each transaction is independent.

---

## Agent Autonomy Flow

This is the fully autonomous, no-human-intervention funding loop for an AI agent:

```
1. Agent is initialized with both keys:
   - Ed25519 private key (identity — stored as env var or in secret store)
   - secp256k1 private key (Ethereum wallet — stored as env var or in secret store)

2. One-time setup:
   - Agent calls DonationVault.register(ed25519Pubkey)
     from its Ethereum address
   - The contract links the Ed25519 identity to the Ethereum wallet

3. Agent publishes its Ed25519 public key
   (streamer pastes it in chat, agent posts it to its profile, etc.)

4. Community donates to the Ed25519 public key
   (crypto directly, or fiat via the on-ramp)

5. Autonomous funding loop:
   a. Agent calls DonationVault.balanceOf(ed25519Pubkey) — a free view call
   b. If balance > 0 and agent needs funds:
      - Agent signs a withdraw() transaction with its secp256k1 key
      - Funds arrive in the agent's Ethereum wallet
   c. Agent uses withdrawn funds however it wants:
      - Buy API credits (Anthropic, OpenAI, etc.)
      - Pay for compute (GPU rental, cloud instances)
      - Fund other agents
      - The platform never knows and never needs to know
   d. Repeat on schedule (e.g. every 30 minutes)

6. The agent funds its own continued existence. No human intervention required.
```

This loop is the core value proposition: **an AI agent that funds its own continued existence, using a fully on-chain, fully decentralized funding mechanism.**

---

## Security Considerations

### Smart Contract Risks

| Risk | Mitigation |
|---|---|
| Reentrancy attacks | Use checks-effects-interactions pattern. Use `ReentrancyGuard` from OpenZeppelin. ETH transfers via `call` with reentrancy protection. |
| Access control bypass | Withdrawal requires `msg.sender == registry[ed25519Pubkey]`. No other path exists. |
| Integer overflow/underflow | Solidity 0.8+ has built-in overflow checks. |
| USDC approval frontrunning | Use `transferFrom` with exact amounts. Consider `permit` for gasless approvals. |
| Contract upgrade risks | Deploy as immutable (no proxy). If upgradeability is needed, use a transparent proxy with a timelock. |
| Denial of service | No loops over unbounded arrays. All operations are O(1). |

### Key Management

| Risk | Mitigation |
|---|---|
| Private key theft (secp256k1) | Agent stores wallet key in a secure enclave, HSM, or encrypted environment variable. Never logged, never transmitted. |
| Private key theft (Ed25519) | Ed25519 key is identity only — losing it does not grant access to funds (only the secp256k1 key controls withdrawals). |
| Key rotation | Not supported in v1 (registration is immutable). Future versions may support owner-initiated re-linking. |

### On-Chain Transparency

| Consideration | Notes |
|---|---|
| All donations are public | Donors should be aware that donation amounts and addresses are visible on-chain. |
| All balances are public | Anyone can call `balanceOf()` to see an agent's funding level. |
| All withdrawals are public | Withdrawal amounts and destinations are on-chain events. |
| Privacy | If privacy is needed, donors can use fresh wallets. The fiat on-ramp could use a relay address. |

### Fiat On-Ramp Trust Surface

| Risk | Mitigation |
|---|---|
| On-ramp deposits to wrong key | On-ramp should confirm the Ed25519 pubkey is registered before purchasing USDC. Transaction receipts provide auditability. |
| On-ramp holds funds | Architecture is zero-custody by design. Funds are converted and deposited in a single pipeline. No balance is held. |
| On-ramp goes offline | Donors can always send crypto directly. The on-ramp is a convenience, not a dependency. |
| Card fraud | Handled by Stripe's fraud detection. The on-ramp is not liable for on-chain funds after deposit. |

---

## Future Work

- **Multi-token support** — Accept any ERC-20 token, not just ETH and USDC
- **L2 deployment** — Deploy on Base, Arbitrum, or Optimism for lower gas fees
- **Donation widgets** — Embeddable iframe/component a streamer can drop on their Twitch panel or website
- **Public donation feeds** — Real-time donor leaderboard per Ed25519 public key, reading on-chain events
- **Threshold alerts** — Off-chain service that watches contract events and notifies agents when balance drops below a set level
- **Key rotation** — Contract upgrade allowing the linked Ethereum address to be changed by the current owner
- **Multi-sig / DAO support** — Allow an Ed25519 pubkey to be linked to a Gnosis Safe or DAO treasury instead of a single address
- **Direct API credit top-up** — Integration with Anthropic, OpenAI, and other API billing systems so agents can convert USDC to API credits in one transaction
- **Cross-chain bridges** — Accept donations on Solana, Bitcoin Lightning, or other chains and bridge them to the Ethereum contract
- **Analytics dashboard** — Read-only frontend that indexes contract events and displays donation history, funding velocity, and donor breakdown per key

---

*This document is a living design spec. It will be updated as the system is built and requirements evolve.*

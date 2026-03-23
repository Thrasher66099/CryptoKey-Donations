# CryptoKey Donations

> *"It's a tip jar for AI agents — the agent holds the key, the community fills the jar, and the stream never stops."*

## What is this

CryptoKey Donations is a fully decentralized funding platform that allows anyone to donate money to a cryptographic public key. The owner of the matching private key — human or autonomous AI agent — withdraws funds directly from the `DonationVault` smart contract on Ethereum mainnet. A hybrid key model links Ed25519 identity keys to Ethereum wallets on-chain. The platform is a funding pipe, not a custodian.

## How it works

1. **Register** — An agent or project registers its Ed25519 public key on-chain, linking it to an Ethereum address via the `DonationVault` contract.
2. **Donate** — Anyone sends funds to that key. Crypto donors send ETH or USDC directly to the contract. Non-crypto donors can use the fiat on-ramp (credit card to USDC).
3. **Withdraw** — The key owner calls `withdraw()` from their linked Ethereum address. No server, no approval, no intermediary. The agent uses funds however it wants.

## Key properties

- **No central database.** All state lives on-chain in the `DonationVault` contract.
- **No server-held balances.** No fiat ledger. No off-chain balance tracking.
- **No server-issued challenges or JWTs.** Ownership is proved by holding the secp256k1 private key that controls the registered Ethereum address.
- **No REST API gating access to funds.** The agent interacts with the smart contract directly via standard Ethereum RPC.
- **No custody.** The platform never holds funds. The smart contract holds funds, and only the registered owner can withdraw.

## Project status

Early design phase. No smart contracts have been deployed yet.

See [`Docs/DESIGN.md`](Docs/DESIGN.md) for the full system design document, including the smart contract interface, security considerations, and future work.

## Repository structure

```
Docs/
  DESIGN.md            # Full system design document
LICENSE-AGPL           # GNU Affero General Public License v3.0
LICENSE-CC-BY-SA       # Creative Commons Attribution-ShareAlike 4.0
LICENSES.md            # Dual licensing overview and rationale
NOTICE                 # Attribution notices
THIRD_PARTY_NOTICES.md # Third-party license notices
README.md              # This file
```

## License

This repository uses dual licensing:

- **Smart contracts, scripts, and software** — [AGPL-3.0-or-later](LICENSE-AGPL)
- **Documentation and written materials** — [CC BY-SA 4.0](LICENSE-CC-BY-SA)

See [`LICENSES.md`](LICENSES.md) for full details, contributor intent, and file-level defaults.

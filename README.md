# Web3 QA Playbook

Structured QA test suites for Web3 applications covering wallet integrations, NFT lifecycles, and blockchain data synchronization.

**Chains:** Ethereum · Polygon
**Wallets:** MetaMask · WalletConnect
**Tools:** Etherscan · PolygonScan · Postman · Browser DevTools

## Contents

| # | Document | Scope | Test Cases |
|---|----------|-------|------------|
| 01 | [MetaMask Checklist](01-metamask-checklist.md) | Connection, transactions, network switching, error handling | 20 |
| 02 | [WalletConnect Matrix](02-walletconnect-matrix.md) | Multi-wallet, multi-platform, session management | 15 |
| 03 | [NFT Test Cases](03-nft-test-cases.md) | Mint, transfer, burn, metadata, marketplace | 18 |
| 04 | [On-chain/Off-chain Sync](04-onchain-offchain-sync.md) | Event listeners, data consistency, failure recovery | 12 |
| 05 | [Cross-Chain Bridge](05-bridge-crosschain.md) | L1↔L2 bridging, deposit/withdraw, status tracking, failure recovery | 15 |
| 06 | [Database Validation](06-database-validation.md) | SQL-based backend verification, balance consistency, event audit | 14 |

**Total: 94 test cases** across 6 core Web3 testing areas.

## About

Written by a QA engineer with hands-on Web3 game testing experience and smart contract development background ([Solana-Defi-Vault](https://github.com/binfengke/Solana-Defi-Vault) — Rust/Anchor).

> "Breaking things in Web3 so users don't have to."

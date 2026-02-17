# Web3 QA Playbook — Design Document

## Purpose

Interview portfolio for Senior QA Engineer (Web3) role at Xsolla. Every file maps directly to a JD requirement.

## Decisions

- **Audience**: Interview showcase (Xsolla hiring managers)
- **Chains**: Ethereum + Polygon
- **Language**: All English
- **Format**: Markdown tables, no diagrams
- **Structure**: Flat, no subdirectories

## Repository Structure

```
web3-qa-playbook/
├── README.md
├── 01-metamask-checklist.md       # 20 test scenarios
├── 02-walletconnect-matrix.md     # 15 test scenarios + compatibility matrix
├── 03-nft-test-cases.md           # 18 test scenarios
├── 04-onchain-offchain-sync.md    # 12 test scenarios + verification methods
└── LICENSE
```

## File Designs

### README.md
- One-line description + tech stack tags (Ethereum, Polygon, MetaMask, WalletConnect)
- Contents table with document names, scope, and test case counts
- Short "About" with link to Solana-Defi-Vault project

### 01-metamask-checklist.md (20 scenarios, MM-01 to MM-20)
- A. Connection (4): first connect, reconnect, reject, not installed
- B. Transactions (4): normal send, reject sign, insufficient balance, gas failure
- C. Network (4): correct network, wrong network, auto-add Polygon, switch interrupted
- D. Account (4): switch account, locked, multi-tab, disconnect/reconnect
- E. Edge Cases (4): concurrent tx, pending timeout, browser refresh, outdated version

### 02-walletconnect-matrix.md (15 scenarios, WC-01 to WC-15)
- A. Connection Flow (4): QR scan, deep link, timeout, cancel
- B. Session Management (4): persistence, dApp disconnect, wallet disconnect, expiry
- C. Transaction Signing (4): normal sign, reject, timeout, app killed
- D. Multi-chain (3): connect on Polygon, switch chain, unsupported chain
- Compatibility matrix: MetaMask Mobile / Trust Wallet / Rainbow / Coinbase Wallet × Desktop / Mobile

### 03-nft-test-cases.md (18 scenarios, NFT-01 to NFT-18)
- A. Minting (5): single, batch, insufficient funds, max supply, ended sale
- B. Transfer (5): normal, invalid address, permission check, network interrupt, not owned
- C. Burn (4): normal, on-chain verify, not owned, UI update
- D. Metadata & Display (4): load metadata, IPFS fallback, metadata refresh, ERC-721 vs ERC-1155
- Extra column: Token Standard (ERC-721 / ERC-1155 / Both)

### 04-onchain-offchain-sync.md (12 scenarios, SYNC-01 to SYNC-12)
- A. Data Consistency (4): success update, failure no-change, token balance match, NFT ownership match
- B. Event Processing (4): event triggers backend, multi-event ordering, listener recovery, high concurrency
- C. Failure & Recovery (4): confirmation delay, block reorg, service restart catchup, RPC failover
- Verification Methods section: Etherscan, Postman, application logs, database queries

## Test Case Table Format (all files)

| Field | Description |
|-------|-------------|
| ID | File prefix + number (MM-01, WC-01, NFT-01, SYNC-01) |
| Scenario | One-line description |
| Preconditions | Wallet state, network, balance, etc. |
| Steps | 3-5 action steps |
| Expected Result | What should happen |
| Priority | High / Medium / Low |
| Chain | Ethereum / Polygon / Both |

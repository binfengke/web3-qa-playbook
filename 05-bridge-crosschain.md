# Cross-Chain Bridge Testing

15 test scenarios covering token bridging between Ethereum and Polygon, including deposit/withdraw flows, fee validation, finality handling, and failure recovery.

## Bridge Concepts

| Term | Description |
|------|-------------|
| **Lock & Mint** | Tokens are locked on the source chain, equivalent tokens are minted on the destination chain |
| **Burn & Unlock** | Tokens are burned on the destination chain, original tokens are unlocked on the source chain |
| **Finality** | The number of block confirmations required before the bridge considers a transaction final |
| **Challenge Period** | A waiting period (e.g., 7 days for Optimistic bridges) before withdrawals can be claimed |
| **Relayer** | An off-chain service that submits proof of the source chain transaction to the destination chain |

## A. Deposit (L1 → L2)

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Direction |
|----|----------|---------------|-------|-----------------|----------|-----------|
| BR-01 | Bridge ETH from Ethereum to Polygon | Wallet connected, sufficient ETH balance on Ethereum | 1. Select source chain: Ethereum 2. Select destination chain: Polygon 3. Enter amount (e.g., 0.1 ETH) 4. Approve and confirm transaction in MetaMask 5. Wait for bridge processing | Transaction confirmed on Ethereum. After finality period, equivalent tokens appear on Polygon. Bridge UI shows correct status updates: Pending → Confirming → Complete | High | L1 → L2 |
| BR-02 | Bridge ERC-20 token (USDC) with approval | Wallet has USDC on Ethereum, no prior approval | 1. Select USDC as bridge token 2. Enter amount 3. MetaMask prompts for ERC-20 approval first 4. Approve spending allowance 5. Confirm the bridge deposit transaction | Two transactions: first approval, then deposit. Both confirmed on-chain. Token balance decreases on Ethereum. After bridge delay, USDC appears on Polygon | High | L1 → L2 |
| BR-03 | Bridge with insufficient balance | Wallet has 0.01 ETH, user tries to bridge 1 ETH | 1. Enter amount exceeding wallet balance 2. Click bridge button | UI shows insufficient balance error before submitting transaction. Bridge button is disabled or displays warning. No MetaMask popup triggered | High | L1 → L2 |
| BR-04 | Bridge minimum amount validation | Bridge has minimum transfer amount (e.g., 0.001 ETH) | 1. Enter amount below minimum (e.g., 0.0001 ETH) 2. Click bridge button | UI displays minimum amount warning. Transaction is not submitted. Error message shows the minimum required amount | Medium | L1 → L2 |
| BR-05 | Gas fee estimation display | Wallet connected to Ethereum | 1. Enter bridge amount 2. Observe gas fee breakdown before confirming | UI displays: bridge fee, destination chain gas fee, estimated total cost, and estimated arrival time. All values are non-zero and reasonable | Medium | L1 → L2 |

## B. Withdraw (L2 → L1)

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Direction |
|----|----------|---------------|-------|-----------------|----------|-----------|
| BR-06 | Bridge tokens from Polygon back to Ethereum | Wallet has tokens on Polygon | 1. Switch network to Polygon 2. Select destination: Ethereum 3. Enter amount 4. Confirm transaction on Polygon 5. Wait for challenge period / finality | Transaction confirmed on Polygon. After required waiting period, tokens are claimable on Ethereum. User must submit a claim transaction on Ethereum to receive tokens | High | L2 → L1 |
| BR-07 | Claim pending withdrawal on L1 | Withdrawal from L2 has passed challenge period | 1. Navigate to bridge transaction history 2. Find the completed withdrawal 3. Click "Claim" button 4. Confirm claim transaction on Ethereum in MetaMask | Claim transaction succeeds on Ethereum. Tokens appear in wallet on Ethereum. Bridge history shows status: Claimed. Token balance on L1 updated correctly | High | L2 → L1 |
| BR-08 | Attempt to claim before challenge period ends | Withdrawal initiated but challenge period not yet complete | 1. Initiate withdrawal from L2 2. Immediately try to claim on L1 | Claim button is disabled or greyed out. UI shows remaining time until claimable. No transaction is submitted. Clear messaging about waiting period | Medium | L2 → L1 |

## C. Status Tracking & History

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Direction |
|----|----------|---------------|-------|-----------------|----------|-----------|
| BR-09 | Bridge transaction status tracking | Bridge transaction submitted | 1. Submit a bridge transaction 2. Navigate to transaction history page 3. Monitor status changes over time | Status progresses through stages: Submitted → Confirming (X/Y confirmations) → Relaying → Complete. Each stage shows timestamp. Source chain tx hash and destination chain tx hash are both displayed and link to correct block explorers | High | Both |
| BR-10 | Transaction history persistence | User has completed multiple bridge transactions | 1. Complete 3+ bridge transactions over time 2. Disconnect wallet 3. Reconnect wallet 4. Check transaction history | All past bridge transactions are displayed with correct details: amount, token, direction, status, timestamps. History is fetched from on-chain data, not just local storage | Medium | Both |

## D. Edge Cases & Failure Recovery

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Direction |
|----|----------|---------------|-------|-----------------|----------|-----------|
| BR-11 | Bridge during source chain congestion | Ethereum network is congested, gas prices are high | 1. Initiate bridge transfer during peak gas prices 2. Observe gas estimation 3. Confirm with high gas fee | Bridge UI warns about high gas fees. If user confirms, transaction may be slow but eventually processes. Bridge handles delayed confirmations without marking as failed | Medium | L1 → L2 |
| BR-12 | Reject bridge transaction in MetaMask | Bridge deposit ready to submit | 1. Enter valid bridge amount 2. Click bridge button 3. Reject in MetaMask popup | Bridge UI returns to input state. No funds are deducted. User can retry. No stuck or pending state in the UI | High | Both |
| BR-13 | Bridge with wrong destination address | User manually enters a recipient address | 1. Enter a valid but wrong destination address 2. Submit bridge transaction | If bridge supports custom recipient: address is validated for correct format. If bridge does not support custom recipient: field is not editable, defaults to connected wallet address | Medium | Both |
| BR-14 | Network switch during bridge process | User switches MetaMask network while bridge UI is open | 1. Connect wallet on Ethereum 2. Start filling bridge form 3. Switch MetaMask to Polygon network manually | Bridge UI detects network change. Either: prompts user to switch back, or auto-updates the source chain field. Does not allow submitting a transaction on the wrong network | High | Both |
| BR-15 | Bridge service downtime handling | Bridge relayer or API is temporarily unavailable | 1. Attempt a bridge transfer while bridge backend is down 2. Observe UI behavior | UI shows clear error: "Bridge service temporarily unavailable" or similar. Does not accept deposits that cannot be relayed. Previously submitted transactions are not affected — they complete once service resumes | Medium | Both |

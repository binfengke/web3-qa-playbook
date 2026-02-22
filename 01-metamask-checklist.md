# MetaMask Wallet Testing Checklist

23 test scenarios covering wallet connection, transaction signing, network management, account handling, edge cases, and mobile browser flows for Ethereum and Polygon networks.

## A. Connection

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain |
|----|----------|---------------|-------|-----------------|----------|-------|
| MM-01 | First-time wallet connection | MetaMask installed, no prior connection to dApp | 1. Open dApp 2. Click "Connect Wallet" 3. Select MetaMask 4. Approve connection in MetaMask popup | dApp displays correct wallet address and network. MetaMask shows dApp in "Connected sites" | High | Both |
| MM-02 | Reconnect after page refresh | MetaMask previously connected to dApp | 1. Connect wallet to dApp 2. Refresh browser page 3. Observe dApp state | dApp auto-reconnects and displays wallet address without requiring manual reconnection | High | Both |
| MM-03 | User rejects connection request | MetaMask installed, connection popup appears | 1. Click "Connect Wallet" 2. Click "Cancel" in MetaMask popup | dApp shows friendly message (e.g., "Wallet connection cancelled"). No crash, no infinite spinner. Connect button remains clickable | High | Both |
| MM-04 | MetaMask not installed | Browser has no MetaMask extension | 1. Open dApp 2. Click "Connect Wallet" 3. Select MetaMask | dApp shows guidance message (e.g., "Please install MetaMask") with link to MetaMask download page. No JavaScript errors in console | Medium | Both |

## B. Transactions

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain |
|----|----------|---------------|-------|-----------------|----------|-------|
| MM-05 | Successful token transfer | Wallet connected, sufficient token balance and ETH/MATIC for gas | 1. Initiate a transfer in dApp 2. Review transaction details in MetaMask popup 3. Click "Confirm" 4. Wait for block confirmation | Transaction confirmed on-chain. dApp updates balance. Transaction hash is displayed and verifiable on Etherscan/PolygonScan | High | Both |
| MM-06 | User rejects transaction signature | Wallet connected, transaction popup appears | 1. Initiate a transfer 2. Click "Reject" in MetaMask popup | dApp shows "Transaction cancelled" message. No state changes. User can retry the action | High | Both |
| MM-07 | Insufficient token balance | Wallet connected, token balance = 0 or less than required amount | 1. Attempt to transfer more tokens than available balance | dApp shows clear error before sending to MetaMask (e.g., "Insufficient balance"). If sent to MetaMask, transaction fails gracefully | High | Both |
| MM-08 | Gas estimation failure | Wallet connected, contract function will revert | 1. Trigger a transaction that will fail on-chain (e.g., minting after sale ended) | MetaMask shows gas estimation error warning. dApp handles the error and shows meaningful message rather than a generic "something went wrong" | Medium | Both |

## C. Network

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain |
|----|----------|---------------|-------|-----------------|----------|-------|
| MM-09 | Connected on correct network | MetaMask set to the network required by dApp | 1. Connect wallet 2. Verify network indicator in dApp | dApp shows correct network name/icon. All features function normally | High | Both |
| MM-10 | Connected on wrong network | MetaMask set to a network not supported by dApp (e.g., BSC) | 1. Switch MetaMask to unsupported network 2. Open dApp or refresh | dApp detects wrong network. Shows clear prompt: "Please switch to Ethereum or Polygon". Core features are disabled until user switches | High | Both |
| MM-11 | Auto-add Polygon network | MetaMask has no Polygon network configured | 1. dApp prompts to add Polygon network 2. MetaMask shows "Add Network" approval 3. User approves | Polygon network added to MetaMask with correct RPC URL, Chain ID (137), currency symbol (MATIC), and block explorer URL | Medium | Polygon |
| MM-12 | Network switch interrupted | MetaMask switching from Ethereum to Polygon | 1. Trigger network switch via dApp 2. Close MetaMask popup before confirming 3. Observe dApp state | dApp remains on previous network. Shows prompt to retry network switch. No broken UI state or stuck spinner | Medium | Both |

## D. Account

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain |
|----|----------|---------------|-------|-----------------|----------|-------|
| MM-13 | Switch account in MetaMask | Wallet connected with Account 1, Account 2 available | 1. Click MetaMask extension 2. Switch to Account 2 3. Observe dApp | dApp detects account change event. Updates displayed address and balance to Account 2. No stale data from Account 1 | High | Both |
| MM-14 | MetaMask is locked | Wallet was connected, user locks MetaMask | 1. Connect wallet 2. Lock MetaMask (timeout or manual) 3. Attempt an action in dApp | dApp detects locked state. Prompts user to unlock MetaMask. Does not show raw errors or crash | Medium | Both |
| MM-15 | Multiple browser tabs connected | Same wallet connected to dApp in 2+ tabs | 1. Open dApp in Tab A and connect wallet 2. Open dApp in Tab B 3. Perform transaction in Tab A 4. Check Tab B | Both tabs reflect the updated state. No conflicting data between tabs. Transaction appears in both | Medium | Both |
| MM-16 | Disconnect and reconnect | Wallet connected to dApp | 1. Disconnect wallet via dApp UI (or via MetaMask "Connected sites") 2. Verify dApp returns to disconnected state 3. Reconnect wallet | Disconnect: dApp clears wallet data, shows "Connect Wallet" button. Reconnect: works normally, previous session data is cleared | Medium | Both |

## E. Edge Cases

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain |
|----|----------|---------------|-------|-----------------|----------|-------|
| MM-17 | Rapid concurrent transactions | Wallet connected, sufficient balance for multiple transactions | 1. Click "Buy" button 3 times rapidly 2. Observe MetaMask popups | dApp either queues transactions with correct nonces or prevents duplicate submissions. No stuck/dropped transactions. All confirmed transactions reflect correctly | High | Both |
| MM-18 | Transaction pending for extended time | Wallet connected, network congested or gas price set very low | 1. Submit transaction with low gas price 2. Wait 5+ minutes with no confirmation | dApp shows pending state with helpful info (e.g., "Transaction is processing on the blockchain..."). Provides option to speed up (resubmit with higher gas) or link to block explorer | Medium | Both |
| MM-19 | Browser refresh during pending transaction | Transaction submitted but not yet confirmed | 1. Submit a transaction 2. Immediately refresh the browser page 3. Observe dApp state after reload | dApp recovers pending transaction state. Shows transaction as pending or checks on-chain status. Does not allow duplicate submission | High | Both |
| MM-20 | Outdated MetaMask version | MetaMask version significantly behind latest release | 1. Install an older version of MetaMask 2. Connect to dApp 3. Attempt a transaction | dApp functions correctly or shows a version warning. No silent failures due to deprecated API calls (e.g., `ethereum.enable()` vs `eth_requestAccounts`) | Low | Both |

## F. Mobile Browser (MetaMask App)

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain |
|----|----------|---------------|-------|-----------------|----------|-------|
| MM-21 | Open dApp in MetaMask mobile in-app browser and connect wallet | MetaMask mobile app installed, test account imported | 1. Open MetaMask mobile app 2. Open in-app browser and navigate to dApp URL 3. Tap "Connect Wallet" | Wallet connection succeeds without extension flow. dApp displays connected address and correct chain in mobile view | High | Both |
| MM-22 | Return to dApp after app switch during pending signature | Transaction/signature request opened in MetaMask mobile browser | 1. Initiate transaction 2. Background MetaMask app or switch to another app 3. Return and confirm/reject request | Request state is preserved after returning. Confirm/reject action is processed once with no duplicate submission or frozen UI | Medium | Both |
| MM-23 | Mobile deep link to block explorer after transaction confirmation | Transaction confirmed in MetaMask mobile browser | 1. Complete transaction 2. Tap transaction hash/explorer link in dApp | Explorer opens correctly on mobile (Etherscan/PolygonScan). Transaction hash matches the submitted transaction | Medium | Both |

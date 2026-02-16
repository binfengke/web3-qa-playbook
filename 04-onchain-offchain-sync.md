# On-chain / Off-chain Synchronization Verification

12 test scenarios covering data consistency, event processing, and failure recovery between blockchain state and application databases. Includes verification methods and tooling.

## Verification Methods

| Method | Tool | Purpose |
|--------|------|---------|
| Block explorer comparison | Etherscan / PolygonScan | Manually compare on-chain state (balances, ownership, tx status) with application UI |
| API assertion | Postman | Call JSON-RPC methods (`eth_call`, `eth_getLogs`, `eth_getTransactionReceipt`) and compare responses with backend API |
| Log monitoring | Application logs / Kibana | Inspect event listener logs for missed, delayed, or duplicate event processing |
| Database query | SQL client / DB admin tool | Directly query off-chain database to verify records match on-chain state |
| Contract state read | Etherscan "Read Contract" tab | Call view functions (e.g., `balanceOf`, `ownerOf`, `totalSupply`) to verify current contract state |

## A. Data Consistency

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain |
|----|----------|---------------|-------|-----------------|----------|-------|
| SYNC-01 | Off-chain updates after successful transaction | Transaction confirmed on-chain | 1. Submit a transaction (e.g., token purchase) 2. Wait for block confirmation 3. Query backend API for updated balance 4. Compare with on-chain balance via Etherscan | Backend database reflects the new balance within expected sync window (e.g., < 30 seconds after confirmation). API response matches on-chain state exactly | High | Both |
| SYNC-02 | Off-chain unchanged after failed transaction | Transaction reverted on-chain | 1. Submit a transaction that will revert (e.g., insufficient allowance) 2. Verify transaction status = "Failed" on block explorer 3. Query backend API for balance | Backend database is unchanged. User balance in application remains the same as before the attempt. No phantom credits or debits | High | Both |
| SYNC-03 | Token balance consistency | User has tokens on-chain | 1. Call `balanceOf(userAddress)` on the smart contract via Etherscan 2. Query the application's balance API endpoint 3. Check the balance displayed in UI | All three values match: contract state = API response = UI display. No rounding errors or decimal precision mismatches (verify 18 decimal vs 6 decimal tokens) | High | Both |
| SYNC-04 | NFT ownership consistency | User owns NFTs on-chain | 1. Call `ownerOf(tokenId)` (ERC-721) or `balanceOf(address, tokenId)` (ERC-1155) on block explorer 2. Check user's inventory in application 3. Transfer NFT externally (e.g., via Etherscan direct contract interaction) 4. Check application inventory again | After external transfer, application detects ownership change and removes NFT from inventory. No stale ownership data. Sync occurs within expected window | High | Both |

## B. Event Processing

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain |
|----|----------|---------------|-------|-----------------|----------|-------|
| SYNC-05 | Smart contract event triggers backend update | Event listener is running, contract emits event (e.g., Transfer) | 1. Execute a contract function that emits an event 2. Check application logs for event receipt 3. Verify backend database was updated based on event data | Event is captured by listener. Log shows event topic, block number, and decoded parameters. Database record is created/updated with correct values from event data | High | Both |
| SYNC-06 | Multiple events in single transaction | A contract function emits 2+ events in one transaction | 1. Execute a batch operation (e.g., batch mint that emits Transfer event per token) 2. Check that all events are processed 3. Verify all corresponding database records | All events from the transaction are processed in correct order. No events are skipped or duplicated. Each event results in the correct database update | High | Both |
| SYNC-07 | Event listener recovery after downtime | Event listener service was stopped for 10+ minutes | 1. Stop the event listener service 2. Execute several transactions on-chain while listener is down 3. Restart the event listener 4. Verify all missed events are processed | Listener catches up by scanning blocks from last processed block number. All events emitted during downtime are processed. Database reaches consistent state with on-chain data | High | Both |
| SYNC-08 | High-concurrency event processing | Multiple users transacting simultaneously | 1. Simulate 50+ concurrent transactions (e.g., during an NFT mint event) 2. Monitor event listener throughput 3. Compare total processed events with total on-chain events | All events are processed without loss. No race conditions in database writes. Event processing queue handles backpressure without dropping messages. Final database state matches on-chain state | Medium | Both |

## C. Failure & Recovery

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain |
|----|----------|---------------|-------|-----------------|----------|-------|
| SYNC-09 | Blockchain confirmation delay | Network is congested, block times are longer than usual | 1. Submit a transaction during high network congestion 2. Observe application behavior while transaction is pending (1-10 minutes) 3. Verify state after eventual confirmation | Application shows pending state during wait. Does not mark transaction as failed prematurely. After confirmation, off-chain state updates correctly. User sees accurate status throughout | High | Both |
| SYNC-10 | Block reorganization (reorg) | A confirmed transaction is removed due to chain reorg | 1. Transaction is confirmed at block N 2. A reorg occurs — block N is replaced 3. Monitor application behavior | Application detects the reorg (block number rollback). Re-evaluates affected transactions. If transaction was dropped, reverts the off-chain state change. If transaction is re-included in new block, maintains the state. Alerts are triggered for operations team | High | Ethereum |
| SYNC-11 | Sync service restart and catchup | Sync service was restarted (e.g., deployment, crash) | 1. Note the last processed block number 2. Restart the sync service 3. Verify it resumes from the correct block 4. Confirm no data gaps | Service resumes from last checkpointed block, not from latest block. Processes all blocks in the gap sequentially. No duplicate processing (idempotent writes). Database state is fully consistent after catchup | High | Both |
| SYNC-12 | RPC node failover | Primary RPC endpoint becomes unresponsive | 1. Block the primary RPC endpoint (e.g., Infura) 2. Monitor application behavior 3. Verify failover to backup RPC (e.g., Alchemy) 4. Restore primary endpoint | Application detects RPC failure within timeout period. Switches to backup RPC node automatically. No data loss during switchover. Logs show failover event. Resumes using primary after it recovers (optional) | Medium | Both |

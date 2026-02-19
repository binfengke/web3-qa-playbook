# Database Validation for Web3 Applications

14 test scenarios covering SQL-based backend verification for blockchain application data integrity. These queries validate that off-chain database records accurately reflect on-chain state and application behavior.

## Common QA Validation Queries

| Purpose | SQL Pattern |
|---------|-------------|
| Verify record exists | `SELECT * FROM table WHERE condition` |
| Check for duplicates | `SELECT column, COUNT(*) FROM table GROUP BY column HAVING COUNT(*) > 1` |
| Cross-table consistency | `SELECT ... FROM table_a JOIN table_b ON key` |
| Data freshness | `SELECT * FROM table WHERE updated_at < NOW() - INTERVAL '5 minutes'` |
| Aggregate validation | `SELECT SUM(amount), COUNT(*) FROM table WHERE condition` |

## A. Transaction Records

| ID | Scenario | Validation Query | Expected Result | Priority |
|----|----------|-----------------|-----------------|----------|
| DB-01 | Transaction record created after on-chain confirmation | `SELECT tx_hash, status, amount, created_at FROM transactions WHERE tx_hash = '0xabc...'` | Record exists with status = 'confirmed', amount matches on-chain value, created_at is within seconds of block timestamp | High |
| DB-02 | Failed transaction does not create false record | `SELECT * FROM transactions WHERE tx_hash = '0xfailed...'` | Either no record exists, or record exists with status = 'failed'. Balance columns remain unchanged from pre-transaction state | High |
| DB-03 | No duplicate transaction records | `SELECT tx_hash, COUNT(*) FROM transactions GROUP BY tx_hash HAVING COUNT(*) > 1` | Query returns zero rows. Each transaction hash appears exactly once in the database | High |
| DB-04 | Transaction amount precision | `SELECT amount, fee, net_amount FROM transactions WHERE id = 123` | Decimal precision matches token standard (6 decimals for USDC, 18 for ETH). net_amount = amount - fee with no rounding errors | High |

## B. User Balance Consistency

| ID | Scenario | Validation Query | Expected Result | Priority |
|----|----------|-----------------|-----------------|----------|
| DB-05 | User balance matches transaction history | `SELECT u.balance, SUM(CASE WHEN t.type='deposit' THEN t.amount WHEN t.type='withdraw' THEN -t.amount END) AS calculated FROM users u JOIN transactions t ON u.id = t.user_id WHERE u.id = 123 GROUP BY u.balance` | u.balance equals calculated sum. No discrepancy between stored balance and transaction-derived balance | High |
| DB-06 | Balance never goes negative | `SELECT id, wallet_address, balance FROM user_balances WHERE balance < 0` | Query returns zero rows. No user has a negative balance in any token | High |
| DB-07 | Balance unchanged after failed transaction | `SELECT balance, updated_at FROM user_balances WHERE user_id = 123` | Balance and updated_at remain the same as before the failed transaction was attempted | Medium |

## C. NFT Ownership Records

| ID | Scenario | Validation Query | Expected Result | Priority |
|----|----------|-----------------|-----------------|----------|
| DB-08 | NFT ownership updates after transfer | `SELECT token_id, owner_address, transferred_at FROM nft_records WHERE token_id = 456` | owner_address matches the new owner from on-chain `ownerOf()` call. transferred_at is within expected sync window of the on-chain transfer block timestamp | High |
| DB-09 | No orphaned NFT records | `SELECT token_id FROM nft_records WHERE owner_address IS NULL OR owner_address = ''` | Query returns zero rows. Every NFT record has a valid owner address | Medium |
| DB-10 | NFT burn removes or flags record | `SELECT token_id, status FROM nft_records WHERE token_id = 789` | After burn: record either deleted, or status = 'burned'. Owner address should not still show previous owner as active holder | Medium |

## D. Event Processing Audit

| ID | Scenario | Validation Query | Expected Result | Priority |
|----|----------|-----------------|-----------------|----------|
| DB-11 | All on-chain events have matching database records | `SELECT e.event_id, e.block_number, e.processed_at FROM blockchain_events e WHERE e.processed_at IS NULL AND e.block_number < (SELECT MAX(block_number) - 10 FROM blockchain_events)` | Query returns zero rows. All events older than 10 blocks have been processed. Any unprocessed events indicate a sync gap | High |
| DB-12 | No duplicate event processing | `SELECT event_id, COUNT(*) FROM blockchain_events GROUP BY event_id HAVING COUNT(*) > 1` | Query returns zero rows. Each blockchain event is processed exactly once (idempotent processing) | High |
| DB-13 | Event processing order preserved | `SELECT event_id, block_number, log_index, processed_at FROM blockchain_events WHERE contract_address = '0xabc...' ORDER BY block_number, log_index` | processed_at timestamps are in ascending order matching block_number and log_index order. No out-of-order processing | Medium |
| DB-14 | Stale data detection | `SELECT table_name, MAX(updated_at) AS last_update FROM sync_metadata GROUP BY table_name HAVING MAX(updated_at) < NOW() - INTERVAL '5 minutes'` | Query returns zero rows during normal operation. Any results indicate a table has not been updated recently and may contain stale data | Medium |

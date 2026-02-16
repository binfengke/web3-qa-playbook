# NFT Lifecycle Test Cases

18 test scenarios covering the complete NFT lifecycle: minting, transferring, burning, and metadata display. Covers both ERC-721 (unique tokens) and ERC-1155 (multi-token) standards for gaming use cases.

## A. Minting

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain | Token Standard |
|----|----------|---------------|-------|-----------------|----------|-------|----------------|
| NFT-01 | Mint a single NFT | Wallet connected, sufficient balance, sale is active | 1. Navigate to mint page 2. Select quantity: 1 3. Click "Mint" 4. Approve transaction in wallet 5. Wait for confirmation | Transaction confirmed. NFT appears in user's inventory/wallet. Token ID is unique. Metadata (name, image, attributes) loads correctly | High | Both | Both |
| NFT-02 | Batch mint multiple NFTs | Wallet connected, sufficient balance for N items, batch minting enabled | 1. Select quantity: 5 2. Click "Mint" 3. Approve transaction (single tx for batch) 4. Wait for confirmation | All 5 NFTs minted in a single transaction. Each has a unique token ID. Total cost = unit price × 5 + gas. All appear in inventory | High | Both | ERC-1155 |
| NFT-03 | Mint with insufficient balance | Wallet connected, token balance less than mint price | 1. Navigate to mint page 2. Click "Mint" | dApp shows "Insufficient balance" error before sending transaction to wallet. If sent to wallet, transaction fails with clear error | High | Both | Both |
| NFT-04 | Mint exceeds max supply | Total supply is at or near the cap | 1. Attempt to mint when supply = max supply 2. Attempt to mint a quantity that would exceed remaining supply | dApp shows "Sold out" or "Only X remaining" message. Smart contract reverts if the transaction is submitted. No tokens minted beyond the cap | High | Both | Both |
| NFT-05 | Mint after sale period ended | Sale end time has passed or sale is paused by admin | 1. Navigate to mint page after sale closes 2. Attempt to click "Mint" | dApp shows "Sale ended" or disables the mint button. If transaction is submitted, smart contract reverts. No tokens minted | Medium | Both | Both |

## B. Transfer

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain | Token Standard |
|----|----------|---------------|-------|-----------------|----------|-------|----------------|
| NFT-06 | Transfer NFT to another wallet | User owns the NFT, recipient address is valid | 1. Select NFT in inventory 2. Click "Transfer" 3. Enter recipient address 4. Approve transaction in wallet 5. Wait for confirmation | NFT removed from sender's inventory. NFT appears in recipient's wallet (verify on block explorer). Ownership updated on-chain | High | Both | Both |
| NFT-07 | Transfer to invalid address | User owns the NFT | 1. Select NFT 2. Click "Transfer" 3. Enter invalid address (e.g., "0x000", random text, empty) | dApp validates address format before submitting. Shows error: "Invalid address". Transaction is not sent to wallet | High | Both | Both |
| NFT-08 | Verify sender loses access after transfer | NFT was transferred in NFT-06 | 1. As the original sender, attempt to transfer the same NFT again 2. Attempt to use/equip the NFT in-game | dApp shows the NFT is no longer owned. Transfer attempt fails. In-game usage is blocked. No stale cache showing the NFT | High | Both | Both |
| NFT-09 | Transfer during network interruption | NFT transfer transaction submitted, network drops | 1. Initiate transfer 2. Approve in wallet 3. Disconnect internet before confirmation 4. Reconnect internet | Transaction either confirms (was already broadcast) or fails. dApp recovers state on reconnection — checks on-chain status and updates UI accordingly. No duplicate or phantom NFTs | Medium | Both | Both |
| NFT-10 | Transfer an NFT you do not own | User does not own the target NFT | 1. Attempt to call transfer function for an NFT owned by another address (via contract interaction or API) | Smart contract reverts with "Not token owner" or equivalent. dApp UI should not allow this action. If attempted via direct contract call, transaction fails | High | Both | Both |

## C. Burn

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain | Token Standard |
|----|----------|---------------|-------|-----------------|----------|-------|----------------|
| NFT-11 | Burn an owned NFT | User owns the NFT, burn function is enabled | 1. Select NFT in inventory 2. Click "Burn" or "Destroy" 3. Confirm warning dialog 4. Approve transaction in wallet 5. Wait for confirmation | NFT is permanently destroyed. Token no longer exists on-chain (verify on block explorer). Removed from user's inventory. Total supply decremented | High | Both | Both |
| NFT-12 | Verify burned NFT on-chain | NFT was burned in NFT-11 | 1. Look up the burned token ID on Etherscan/PolygonScan 2. Query ownerOf() for ERC-721 or balanceOf() for ERC-1155 | Block explorer shows burn transaction. ownerOf() reverts or returns zero address. balanceOf() returns 0. Token metadata is no longer retrievable | High | Both | Both |
| NFT-13 | Burn an NFT you do not own | User does not own the target NFT | 1. Attempt to call burn function for another user's NFT | Smart contract reverts. Transaction fails. The NFT remains in the rightful owner's possession | High | Both | Both |
| NFT-14 | UI update after burn | NFT was just burned | 1. Burn an NFT 2. Observe inventory page 3. Refresh the page | NFT is immediately removed from inventory after transaction confirmation. After refresh, NFT is still gone. Inventory count is decremented. No ghost items | Medium | Both | Both |

## D. Metadata & Display

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain | Token Standard |
|----|----------|---------------|-------|-----------------|----------|-------|----------------|
| NFT-15 | Metadata loads correctly | NFT is minted and owned by user | 1. Open NFT detail page 2. Verify name, description, image, and attributes | All metadata fields match the token URI data. Image renders correctly. Attributes (e.g., rarity, level, stats) display with correct values and formatting | High | Both | Both |
| NFT-16 | IPFS image fails to load | NFT image is hosted on IPFS, IPFS gateway is slow or down | 1. Open NFT detail page when IPFS gateway is unreachable 2. Observe image area | dApp shows a placeholder image or loading skeleton. No broken image icon. Retries loading or shows "Image unavailable" message. Other metadata still displays | Medium | Both | Both |
| NFT-17 | Metadata updates reflected in UI | NFT metadata is updated on-chain or off-chain (e.g., game item leveled up) | 1. Update NFT metadata (e.g., change attribute value) 2. Refresh NFT detail page 3. Verify new values | Updated metadata displays correctly. Cache is invalidated. Old values are not shown. If on-chain metadata, verify the update transaction on block explorer | Medium | Both | Both |
| NFT-18 | ERC-721 vs ERC-1155 display differences | dApp supports both token standards | 1. View an ERC-721 NFT (unique item, quantity = 1) 2. View an ERC-1155 NFT (stackable item, quantity = N) 3. Compare display | ERC-721: shows as unique item with token ID, no quantity. ERC-1155: shows quantity owned (e.g., "x5"), supports partial transfer. Both display metadata correctly | Medium | Both | Both |

# WalletConnect Testing Matrix

15 test scenarios covering connection flows, session management, transaction signing, and multi-chain support. Includes a cross-platform compatibility matrix for wallet × device combinations.

## A. Connection Flow

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain |
|----|----------|---------------|-------|-----------------|----------|-------|
| WC-01 | QR code scan connection | dApp on desktop browser, mobile wallet installed | 1. Click "Connect Wallet" in dApp 2. Select WalletConnect 3. QR code is displayed 4. Scan QR code with mobile wallet 5. Approve connection in wallet | dApp shows connected state with correct address. Mobile wallet shows active session with dApp name and icon | High | Both |
| WC-02 | Deep link connection (mobile) | dApp open in mobile browser, wallet app installed on same device | 1. Click "Connect Wallet" in mobile dApp 2. Select WalletConnect 3. Choose wallet from list 4. Wallet app opens automatically 5. Approve connection | Wallet app opens via deep link. After approval, user is returned to dApp in browser. Connection is established | High | Both |
| WC-03 | Connection request timeout | dApp shows QR code, user does not scan | 1. Click "Connect Wallet" 2. QR code is displayed 3. Wait without scanning for 60+ seconds | dApp shows timeout message (e.g., "Connection timed out. Please try again"). QR code is invalidated. User can generate a new one | Medium | Both |
| WC-04 | User cancels connection | dApp shows QR code or wallet shows approval prompt | 1. Click "Connect Wallet" 2. QR code appears 3. Scan QR code 4. Click "Cancel" in wallet app | dApp returns to disconnected state. No error, no crash. "Connect Wallet" button remains functional | High | Both |

## B. Session Management

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain |
|----|----------|---------------|-------|-----------------|----------|-------|
| WC-05 | Session persists after browser close | Wallet connected via WalletConnect | 1. Connect wallet 2. Close browser completely 3. Reopen browser and navigate to dApp | dApp reconnects using saved session. Wallet address and balance display correctly without requiring new QR scan | High | Both |
| WC-06 | Disconnect from dApp side | Active WalletConnect session | 1. Click "Disconnect" in dApp UI 2. Check wallet app session list | dApp returns to disconnected state. Wallet app removes the session from active connections. No orphaned sessions | High | Both |
| WC-07 | Disconnect from wallet side | Active WalletConnect session | 1. Open wallet app 2. Navigate to active sessions 3. Disconnect the dApp session 4. Return to dApp in browser | dApp detects disconnection. Updates UI to show disconnected state. Prompts user to reconnect if they attempt an action | High | Both |
| WC-08 | Session expiry | Active WalletConnect session, session TTL configured | 1. Connect wallet 2. Leave session idle until expiry 3. Attempt an action in dApp | dApp detects expired session. Shows message (e.g., "Session expired. Please reconnect"). New connection flow works normally | Medium | Both |

## C. Transaction Signing

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain |
|----|----------|---------------|-------|-----------------|----------|-------|
| WC-09 | Successful transaction signing | Active session, sufficient balance | 1. Initiate transaction in dApp 2. Signing request appears in wallet app 3. Review details in wallet 4. Approve transaction | Transaction submitted to blockchain. dApp shows confirmation with tx hash. Balance updates after block confirmation | High | Both |
| WC-10 | User rejects signing in wallet | Active session, signing request sent | 1. Initiate transaction in dApp 2. Signing request appears in wallet 3. Click "Reject" in wallet | dApp receives rejection. Shows user-friendly message. No state changes. User can retry | High | Both |
| WC-11 | Signing request timeout | Active session, signing request sent but user ignores | 1. Initiate transaction in dApp 2. Do not respond in wallet app for 60+ seconds | dApp shows timeout message. Transaction is not submitted. User can retry the action | Medium | Both |
| WC-12 | Wallet app killed during signing | Active session, signing request pending | 1. Initiate transaction in dApp 2. Signing request sent to wallet 3. Force-close wallet app 4. Reopen wallet app | dApp handles the interrupted request gracefully (timeout or retry prompt). Wallet app does not show stale signing request after relaunch | Medium | Both |

## D. Multi-chain

| ID | Scenario | Preconditions | Steps | Expected Result | Priority | Chain |
|----|----------|---------------|-------|-----------------|----------|-------|
| WC-13 | Connect with Polygon selected | Wallet configured with Polygon network | 1. Set wallet to Polygon network 2. Connect to dApp via WalletConnect 3. Verify chain in dApp | dApp detects Polygon (Chain ID 137). Displays Polygon network indicator. Shows MATIC balance | High | Polygon |
| WC-14 | Switch chain after connection | Active session on Ethereum | 1. Connect on Ethereum 2. dApp requests chain switch to Polygon 3. Approve in wallet | Wallet switches to Polygon. dApp updates network indicator and balance. Existing session remains active | High | Both |
| WC-15 | dApp requests unsupported chain | Active session, dApp requests a chain not in wallet | 1. Connect wallet 2. dApp requests switch to an unsupported chain (e.g., Avalanche) | Wallet shows error or ignores the request. dApp handles rejection and shows message (e.g., "Network not supported. Please use Ethereum or Polygon") | Medium | Both |

## Compatibility Matrix

Test coverage across wallet × platform combinations:

|  | MetaMask Mobile | Trust Wallet | Rainbow | Coinbase Wallet |
|--|-----------------|-------------|---------|-----------------|
| **Desktop browser dApp** | ✅ | ✅ | ✅ | ✅ |
| **Mobile browser dApp** | ✅ | ✅ | ⬜ | ⬜ |

- ✅ Tested
- ⬜ Not yet tested (planned)

### Notes

- Desktop browser testing performed on Chrome and Firefox
- Mobile browser testing performed on iOS Safari and Android Chrome
- QR code scanning tested with rear and front cameras
- Deep link flow only applicable to mobile browser → wallet app scenarios

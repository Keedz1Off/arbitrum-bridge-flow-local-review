# Glossary

This file contains important words used in this Arbitrum bridge review.

The goal is to keep the key audit vocabulary in one place.

## Core Terms

| Term | Simple Meaning | Where It Matters |
|---|---|---|
| bridge | System that moves value/messages between chains | Full deposit and withdrawal flow |
| L1 | Ethereum / source settlement chain | L1 Gateway, Inbox, Outbox |
| L2 | Arbitrum chain | L2 Gateway, token mint/release |
| gateway | Bridge contract responsible for token flow | `outboundTransfer(...)`, `finalizeInboundTransfer(...)` |
| counterpart gateway | Expected gateway on the other chain | Auth boundary |
| escrow | Tokens locked on the bridge contract | `outboundEscrowTransfer(...)` |
| lock | Keep tokens inside escrow | Deposit accounting |
| release | Send escrowed tokens back to the user | Withdrawal finalization |
| burn | Destroy tokens before withdrawal | L2 withdrawal flow |
| mint | Create tokens after deposit finalization | L2 deposit finalization |
| retryable ticket | Arbitrum L1 -> L2 message mechanism | `createRetryableTicket(...)` |
| inbox | Arbitrum contract that creates L1 -> L2 messages | Deposit message creation |
| outbox | Arbitrum contract used for L2 -> L1 execution/proofs | Withdrawal flow |
| calldata | Encoded function call data | Message construction |
| payload | Encoded cross-chain message data | `getOutboundCalldata(...)` |
| aliasing | Address transformation used for L1 -> L2 calls | Arbitrum message sender logic |
| actualReceived | Real balance increase after token transfer | Fee-on-transfer accounting |
| fee-on-transfer | Token that takes a fee during transfer | Escrow amount mismatch |
| refund address | Address that receives unused retryable funds | Retryable ticket creation |

## Security Words

| Term | Simple Meaning |
|---|---|
| invariant | Rule that must always stay true |
| broken invariant | Rule that was violated |
| impact | What bad thing can happen |
| root cause | Why the bug exists |
| spoofed message | Fake message pretending to be authentic |
| wrong gateway | Message is finalized by the wrong bridge contract |
| wrong token | L1 token does not match the intended L2 token |
| wrong recipient | Funds are credited/released to the wrong address |
| wrong amount | Credited amount does not match locked/burned amount |
| unbacked tokens | Tokens minted/released without real backing |
| stuck funds | Funds are locked but the user cannot receive them |
| finalization | Destination-side execution of the bridge message |

## Simple English Phrases

```text
L1 escrowed amount must equal L2 minted amount.
Only an authentic bridge message can finalize a deposit.
The counterpart gateway must be the expected gateway.
The selected token must be the intended token.
The recipient must be preserved through the message.
```

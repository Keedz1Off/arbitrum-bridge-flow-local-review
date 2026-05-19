# Arbitrum Bridge Flow Local Review

This repository contains my local study and security-oriented review of an Arbitrum-style bridge flow.

The goal of this repository is to document how I studied the bridge architecture, deposit flow, withdrawal flow, and important function-level security concepts.

This is an educational portfolio project. It is not an official audit of Arbitrum or any production deployment.

---

## Repository Overview

This repository is organized into several parts:

```text
deposit-flow/
```

Function-by-function notes for the L1 -> L2 deposit flow.

```text
withdrawal-flow/
```

Function-by-function notes for the L2 -> L1 withdrawal flow.

```text
concepts/
```

Separate explanations of important bridge concepts, such as Arbitrum address aliasing.

```text
breaksync/
```

Placeholder folder for a future BreakSync-style manual analysis.

---

## Bridge Flow Overview

Deposit direction:

```text
User
  |
  v
L1 Gateway
  |
  | lock / escrow token
  v
Inbox / Retryable Ticket
  |
  | L1 -> L2 message
  v
L2 Gateway
  |
  | finalize / mint token
  v
L2 Recipient
```

Withdrawal direction:

```text
L2 User
  |
  v
L2 Gateway
  |
  | burn / lock token
  v
Outbox Message
  |
  | L2 -> L1 proof
  v
L1 Gateway
  |
  | finalize / release token
  v
L1 Recipient
```

---

## Study Process

I studied the bridge in several stages:

1. First, I studied the high-level architecture of the bridge.
2. Then I traced the full deposit flow.
3. Then I traced the full withdrawal flow.
4. After that, I reviewed important functions one by one.
5. Finally, I organized the notes into this repository.

I partially used AI as a writing and organization assistant while preparing the notes, but the goal of this repository is to show my own learning process, flow tracing, and security reasoning.

---

## Repository Structure

```text
arbitrum-bridge-flow-local-review/
|
├── README.md
|
├── deposit-flow/
│   ├── 01-outboundTransfer.md
│   ├── 02-outboundEscrowTransfer.md
│   ├── 03-getOutboundCalldata.md
│   ├── 04-createRetryableTicket.md
│   ├── 05-AbsInbox-createRetryableTicket.md
│   ├── 06-finalizeInboundTransfer.md
│   └── 07-inboundEscrowTransfer-or-mint.md
|
├── withdrawal-flow/
│   ├── 01-outboundTransfer-or-withdraw.md
│   ├── 02-burn-or-lock.md
│   ├── 03-getOutboundCalldata.md
│   ├── 04-createOutboundTx.md
│   ├── 05-finalizeInboundTransfer-or-finalizeWithdrawal.md
│   └── 06-inboundEscrowTransfer-or-release.md
|
├── concepts/
│   └── address-aliasing.md
|
└── breaksync/
    └── README.md
```

---

## Current Scope

The current focus of the repository is:

- bridge flow overview
- deposit flow notes
- withdrawal flow notes
- function-level explanations
- important bridge concepts

The `breaksync/` folder is reserved for a future manual BreakSync-style analysis, where each function will be reviewed separately in more depth.

---

## Disclaimer

This repository is for educational and portfolio purposes only.

It is not an official audit, and it should not be treated as a security assessment of any production deployment.

# Function Review: createRetryableTicket(...)

## 1. Function Code

```solidity
// Paste the exact createRetryableTicket(...) source code here.
//
// Example structure:
//
// function createRetryableTicket(
//     address _to,
//     uint256 _l2CallValue,
//     uint256 _maxSubmissionCost,
//     address _excessFeeRefundAddress,
//     address _callValueRefundAddress,
//     uint256 _gasLimit,
//     uint256 _maxFeePerGas,
//     bytes calldata _data
// ) external payable returns (uint256) {
//     ...
// }
```

---

## 2. Function Purpose

`createRetryableTicket(...)` creates the L1 -> L2 message that will execute the
deposit finalization on Arbitrum.

In simple terms:

```text
This function sends the prepared deposit calldata from L1 to L2.
```

It is critical because it connects L1 accounting with L2 execution.

---

## 3. Critical Parameters

- `_to`: L2 target contract
- `_data`: calldata executed on L2
- `_l2CallValue`: ETH value sent to L2 call
- `_maxSubmissionCost`: cost for submitting retryable ticket
- `_gasLimit`: gas budget for L2 execution
- `_maxFeePerGas`: max L2 gas price
- `_excessFeeRefundAddress`: receives excess fee refund
- `_callValueRefundAddress`: receives call value refund if execution fails

---

## 4. Trusted / Untrusted Input

Trusted or semi-trusted:

- configured inbox
- expected L2 gateway target
- calldata built by trusted gateway logic

Untrusted:

- user-controlled gas parameters
- refund addresses
- user-influenced calldata
- user-provided call value assumptions

Important audit idea:

```text
A correct deposit can still fail if the retryable ticket is built with unsafe execution parameters.
```

---

## 5. Called Inside

Typical call structure:

```text
outboundTransfer(...)
└── createRetryableTicket(...)
    └── AbsInbox._createRetryableTicket(...)
```

Security meaning:

```text
This is the cross-chain message creation boundary.
```

---

## 6. Invariants

- Retryable ticket must target the correct L2 gateway.
- Calldata must contain the correct `amount`, `token`, and `recipient`.
- The ticket must have enough execution budget.
- Refund addresses must not create unsafe value redirection.
- The message must not be replayable outside the bridge system.

---

## 7. Message Consistency

Values to trace:

```text
verified L1 deposit
-> encoded calldata
-> retryable ticket
-> L2 execution
-> finalizeInboundTransfer(...)
```

Main question:

```text
Does the retryable ticket execute the exact calldata that represents the real L1 deposit?
```

---

## 8. Security Risks

### Risk 1: Wrong L2 target

If `_to` is incorrect, the message may execute on the wrong L2 contract.

Impact:

```text
Invalid finalization or lost deposit execution.
```

### Risk 2: Corrupted calldata

If `_data` contains wrong `amount`, `token`, or `recipient`, L2 may finalize an
incorrect deposit.

Impact:

```text
Broken token accounting or recipient substitution.
```

### Risk 3: Insufficient gas

If gas parameters are too low, the retryable ticket may fail.

Impact:

```text
Stuck or delayed deposit.
```

### Risk 4: Unsafe refund addresses

If refund addresses are attacker-controlled in an unsafe way, value may be
redirected.

Impact:

```text
Execution value/refund redirection.
```

---

## 9. Audit Questions

- Who controls the L2 target address?
- Is `_to` always the expected L2 gateway?
- What calldata is passed into the retryable ticket?
- Can user input corrupt `_data`?
- Are gas parameters validated?
- Can low gas cause stuck deposits?
- Who controls refund addresses?
- Can refund logic redirect value unexpectedly?
- Is the retryable ticket uniquely associated with this deposit?

---

## 10. Audit Conclusion

`createRetryableTicket(...)` is the L1 -> L2 message creation boundary.

Most important invariant:

```text
Retryable ticket calldata must represent the verified L1 deposit.
```

Main risk:

```text
The bridge creates a valid cross-chain message with invalid or unsafe parameters.
```

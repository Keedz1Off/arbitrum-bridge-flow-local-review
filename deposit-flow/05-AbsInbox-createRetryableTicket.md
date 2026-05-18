
# Function Review: AbsInbox._createRetryableTicket(...)

## 1. Function Code

```solidity
// Paste the exact AbsInbox._createRetryableTicket(...) source code here.
//
// Example structure:
//
// function _createRetryableTicket(
//     address _to,
//     uint256 _l2CallValue,
//     uint256 _maxSubmissionCost,
//     address _excessFeeRefundAddress,
//     address _callValueRefundAddress,
//     uint256 _gasLimit,
//     uint256 _maxFeePerGas,
//     uint256 _amount,
//     bytes calldata _data
// ) internal returns (uint256) {
//     ...
// }
```

---

## 2. Function Purpose

`AbsInbox._createRetryableTicket(...)` is lower-level Arbitrum Inbox logic that
creates or records the retryable ticket for L2 execution.

In simple terms:

```text
This is the lower-level system step that puts the L1 -> L2 message into Arbitrum's messaging flow.
```

It is important because it defines how the L2 message is submitted, paid for, and
represented inside the bridge system.

---

## 3. Critical Parameters

- `_to`: L2 target
- `_data`: L2 execution calldata
- `_l2CallValue`: value used in L2 call
- `_maxSubmissionCost`: retryable submission cost
- `_gasLimit`: L2 execution gas limit
- `_maxFeePerGas`: max gas price
- `_excessFeeRefundAddress`: excess fee refund receiver
- `_callValueRefundAddress`: call value refund receiver
- message sender context

---

## 4. Trusted / Untrusted Input

Trusted or semi-trusted:

- Arbitrum Inbox implementation
- bridge system assumptions
- configured retryable ticket mechanism

Untrusted or risky:

- caller-provided target
- caller-provided calldata
- caller-provided gas values
- caller-provided refund addresses

Important audit idea:

```text
Even if the Inbox works correctly, the caller must provide correct message parameters.
```

---

## 5. Called Inside

Typical call structure:

```text
createRetryableTicket(...)
└── AbsInbox._createRetryableTicket(...)
```

Security meaning:

```text
This is the low-level retryable ticket boundary.
```

---

## 6. Invariants

- The retryable ticket must preserve the intended L2 target.
- The retryable ticket must preserve the intended calldata.
- Payment and gas values must be sufficient for message creation.
- Refund behavior must follow the expected Arbitrum rules.
- L2 sender semantics must be understood correctly.
- If the sender is an L1 contract, address aliasing must be considered.

---

## 7. Address Aliasing Consideration

In Arbitrum, when an L1 contract sends a message to L2, the L2 side may see an
aliased version of the L1 contract address.

Simple idea:

```text
L1 contract address -> L2 aliased sender address
```

Audit question:

```text
Does the L2 finalize function check the expected aliased sender?
```

This matters because incorrect sender assumptions can break cross-chain
authentication.

---

## 8. Security Risks

### Risk 1: Wrong sender assumption on L2

If L2 expects the raw L1 address but Arbitrum provides the aliased address, auth
checks may fail or be implemented incorrectly.

Impact:

```text
Broken cross-chain authentication.
```

### Risk 2: Wrong target or calldata

If the caller provides incorrect `_to` or `_data`, the Inbox may correctly create
a retryable ticket that performs the wrong action.

Impact:

```text
Correct system execution with incorrect bridge parameters.
```

### Risk 3: Insufficient payment or gas

If payment/gas values are insufficient, retryable creation or execution may fail.

Impact:

```text
Stuck or delayed cross-chain message.
```

### Risk 4: Unsafe refund handling

Incorrect refund addresses may redirect value or create unexpected behavior.

Impact:

```text
Refund/value redirection risk.
```

---

## 9. Audit Questions

- What exact L2 target is passed into `_createRetryableTicket(...)`?
- What calldata is passed?
- Are gas and fee values sufficient?
- Who controls refund addresses?
- What will the L2 sender be?
- Is the sender an EOA or L1 contract?
- If the sender is an L1 contract, does address aliasing apply?
- Does L2 auth check the correct sender form?
- Can incorrect parameters still create a valid but dangerous retryable ticket?

---

## 10. Audit Conclusion

`AbsInbox._createRetryableTicket(...)` is the low-level retryable ticket creation
step.

Most important invariant:

```text
The retryable ticket must preserve the correct target, calldata, and sender assumptions.
```

Main risk:

```text
The Inbox correctly creates a message, but the bridge provided unsafe parameters.
```

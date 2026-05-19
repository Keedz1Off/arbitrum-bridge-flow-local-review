# Function Review: createRetryableTicket(...)

## Function Code

```solidity
function createRetryableTicket(
    address to,
    uint256 l2CallValue,
    uint256 maxSubmissionCost,
    address excessFeeRefundAddress,
    address callValueRefundAddress,
    uint256 gasLimit,
    uint256 maxFeePerGas,
    bytes calldata data
) external payable whenNotPaused onlyAllowed returns (uint256) {
    return _createRetryableTicket(
        to,
        l2CallValue,
        maxSubmissionCost,
        excessFeeRefundAddress,
        callValueRefundAddress,
        gasLimit,
        maxFeePerGas,
        msg.value,
        data
    );
}
```

---

## Function Explanation

`createRetryableTicket(...)` creates the L1 -> L2 retryable ticket.

In the deposit flow, this is the step where prepared calldata is submitted to Arbitrum so it can later execute on L2.

Main idea:

```text
The retryable ticket transports the deposit message from L1 to L2.
```

This function is important because a valid retryable ticket can still be dangerous if it targets the wrong L2 contract or contains wrong calldata.

---

## Important Logic Notes

### L2 Target

The retryable ticket must target the correct L2 gateway.

```text
wrong L2 target = wrong execution path
```

If the target is wrong, the message may fail or execute unintended logic.

---

### Calldata Passed to L2

The calldata passed into the retryable ticket should already contain verified deposit values.

Important values:

- amount
- token
- recipient
- finalize selector

The ticket should not carry corrupted or user-overridden bridge data.

---

### Gas and Refund Parameters

Retryable tickets depend on execution budget and refund addresses.

Important values may include:

- gas limit
- max fee per gas
- max submission cost
- excess fee refund address
- call value refund address

If these values are unsafe, the deposit may become delayed, fail, or redirect value unexpectedly.

---

## Invariants

- Retryable ticket must target the correct L2 gateway.
- Retryable calldata must match the verified L1 deposit.
- Retryable execution budget must be sufficient for finalization.
- Refund handling must not corrupt bridge accounting.
- The ticket must not allow the same deposit to be finalized twice.

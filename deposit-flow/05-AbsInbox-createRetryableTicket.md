# Function Review: AbsInbox._createRetryableTicket(...)

## Function Code

```solidity
function _createRetryableTicket(
    address to,
    uint256 l2CallValue,
    uint256 maxSubmissionCost,
    address excessFeeRefundAddress,
    address callValueRefundAddress,
    uint256 gasLimit,
    uint256 maxFeePerGas,
    uint256 amount,
    bytes calldata data
) internal returns (uint256) {
    // Ensure the user's deposit alone will make submission succeed.
    // In case of native token having non-18 decimals: 'amount' is denominated in native token's decimals. All other
    // value params - l2CallValue, maxSubmissionCost and maxFeePerGas are denominated in child chain's native 18 decimals.
    uint256 amountToBeMintedOnL2 = _fromNativeTo18Decimals(amount);
    if (amountToBeMintedOnL2 < (maxSubmissionCost + l2CallValue + gasLimit * maxFeePerGas)) {
        revert InsufficientValue(
            maxSubmissionCost + l2CallValue + gasLimit * maxFeePerGas, amountToBeMintedOnL2
        );
    }

    // if a refund address is a contract, we apply the alias to it
    // so that it can access its funds on the L2
    // since the beneficiary and other refund addresses don't get rewritten by arb-os
    if (AddressUpgradeable.isContract(excessFeeRefundAddress)) {
        excessFeeRefundAddress = AddressAliasHelper.applyL1ToL2Alias(excessFeeRefundAddress);
    }
    if (AddressUpgradeable.isContract(callValueRefundAddress)) {
        // this is the beneficiary. be careful since this is the address that can cancel the retryable in the L2
        callValueRefundAddress = AddressAliasHelper.applyL1ToL2Alias(callValueRefundAddress);
    }

    // gas limit is validated to be within uint64 in unsafeCreateRetryableTicket
    return _unsafeCreateRetryableTicket(
        to,
        l2CallValue,
        maxSubmissionCost,
        excessFeeRefundAddress,
        callValueRefundAddress,
        gasLimit,
        maxFeePerGas,
        amount,
        data
    );
}
```

---

## Function Explanation

`AbsInbox._createRetryableTicket(...)` is the lower-level Arbitrum Inbox step that creates or records the retryable ticket.

This function is closer to the message system than to token accounting.

Main idea:

```text
The Inbox creates the L1 -> L2 message, but the caller must provide correct bridge parameters.
```

Even if the Inbox logic works correctly, the bridge can still be unsafe if it passes the wrong target, calldata, gas values, or refund addresses.

---

## Important Logic Notes

### Message Creation

This function inserts the retryable ticket into the Arbitrum messaging flow.

The important bridge values are already packed before this step.

So the key issue is whether the values passed into this function are already correct.

---

### Sender Semantics and Address Aliasing

For Arbitrum, if an L1 contract sends a message to L2, the L2 side may see an aliased version of the L1 contract address.

Simple idea:

```text
L1 contract sender -> aliased L2 sender
```

If the L2 finalize function checks the wrong sender form, authentication can break.

---

### Refund and Payment Logic

Retryable ticket creation involves submission cost, gas funding, and refund addresses.

This does not directly decide token amount, but it can affect whether the L2 message executes successfully and where unused value is refunded.

---

## Invariants

- The retryable ticket must preserve the intended L2 target.
- The retryable ticket must preserve the intended calldata.
- L2 sender assumptions must match Arbitrum aliasing rules.
- Gas and payment values must be sufficient for message creation/execution.
- Refund behavior must not redirect value in an unintended way.

# Function Review: getOutboundCalldata(...)

## Function Code

```solidity
// Paste the full getOutboundCalldata(...) function code here.
// Use the exact code from the contract version you are reviewing.

function getOutboundCalldata(
    // paste exact parameters here
) public view returns (bytes memory) {
    // paste exact function body here
}
```

---

## Function Explanation

`getOutboundCalldata(...)` builds the calldata / payload that will be sent from L1 to L2.

This function usually runs after the bridge has selected the token path and prepared the deposit values.

The main purpose is to encode the values that the L2 gateway will later decode and use during finalization.

Main idea:

```text
Calldata must represent the real L1 deposit.
```

If wrong values are encoded here, the L2 side may execute normally but credit the wrong amount, token, or recipient.

---

## Important Logic Notes

### Encoding Step

If the function uses something like:

```solidity
abi.encode(...);
```

or:

```solidity
abi.encodeWithSelector(...);
```

this is the point where bridge values become message data.

Important values usually include:

- token
- sender
- recipient
- amount
- extra data

The values must be correct before encoding.

---

### Amount in Calldata

The encoded amount should match the amount that is actually backed by L1 escrow.

Important distinction:

```text
input amount != always actual received amount
```

If the bridge received less than the user-supplied amount but encodes the original amount, L2 may credit more than L1 holds.

---

### Encode / Decode Match

The format used here must match the format expected by the L2 finalize function.

```text
L1 encode format = L2 decode format
```

If these formats differ, L2 may decode the wrong amount, token, or recipient.

---

## Invariants

- Encoded calldata must represent the real L1 deposit.
- Encoded amount must match the escrowed / actual received amount.
- Encoded recipient must match the intended L2 recipient.
- Encoded token must match the correct token mapping.
- L1 encoding must match L2 decoding.
- Extra data must not override critical values in an unsafe way.

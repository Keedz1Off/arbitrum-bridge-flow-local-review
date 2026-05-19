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

`getOutboundCalldata(...)` builds the calldata / payload for the L2 -> L1 withdrawal message.

This calldata will later be decoded on L1 during withdrawal finalization.

Main idea:

```text
The L1 finalization calldata must represent the real L2 withdrawal.
```

---

## Important Logic Notes

### Encoding Step

If the function uses `abi.encode(...)` or `abi.encodeWithSelector(...)`, this is where withdrawal values become message data.

Important values usually include:

- L1 token
- L2 token
- sender
- recipient
- amount
- extra data

---

### Amount in Calldata

The encoded amount should match the amount actually burned or locked on L2.

If the encoded amount is larger than the burned/locked amount, L1 may release too much.

---

### Encode / Decode Match

The L2 encode format must match the L1 decode format.

```text
L2 encode format = L1 decode format
```

If the formats differ, L1 may decode the wrong amount, token, or recipient.

---

## Invariants

- Encoded calldata must represent the real L2 withdrawal.
- Encoded amount must match the burned / locked amount.
- Encoded recipient must match the intended L1 recipient.
- Encoded token must match the correct L1 token mapping.
- L2 encoding must match L1 decoding.
- Extra data must not override critical values in an unsafe way.

# Function Review: createOutboundTx(...)

## Function Code

```solidity
// Paste the full createOutboundTx(...) function code here.
// Use the exact code from the contract version you are reviewing.

function createOutboundTx(
    // paste exact parameters here
) internal returns (uint256) {
    // paste exact function body here
}
```

---

## Function Explanation

`createOutboundTx(...)` creates the L2 -> L1 message for withdrawal finalization.

This function sends the prepared withdrawal calldata toward L1.

Main idea:

```text
The outbound message must carry the verified withdrawal data to the correct L1 gateway.
```

---

## Important Logic Notes

### L1 Target

The outbound transaction must target the correct L1 gateway or finalization contract.

Wrong target can make the withdrawal fail or execute unintended logic.

---

### Calldata Passed to L1

The calldata should contain the verified withdrawal values:

- token
- recipient
- amount
- finalize selector

If calldata is wrong, L1 may release the wrong asset, amount, or recipient.

---

### Message Identity

The withdrawal message should be uniquely identifiable by the bridge system.

This matters for replay protection and finalization tracking.

---

## Invariants

- Outbound message must target the correct L1 gateway.
- Outbound calldata must match the verified L2 withdrawal.
- Message identity must prevent duplicate finalization.
- L1 release must only happen through a valid L2 -> L1 message.

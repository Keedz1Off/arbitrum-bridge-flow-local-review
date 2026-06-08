# Function Review: createOutboundTx(...)

## Function Code

```solidity
function createOutboundTx(
    address _from,
    uint256, /* _tokenAmount */
    bytes memory _outboundCalldata
) internal virtual returns (uint256) {
    // We make this function virtual since outboundTransfer logic is the same for many gateways
    // but sometimes (ie weth) you construct the outgoing message differently.

    // exitNum incremented after being included in _outboundCalldata
    exitNum++;
    return
        sendTxToL1(
            // default to sending no callvalue to the L1
            0,
            _from,
            counterpartGateway,
            _outboundCalldata
        );
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

### Main Invariant 1

```text
Outbound message must target the correct L1 gateway.
```

### Main Invariant 2

```text
Outbound calldata must match the verified L2 withdrawal.
```

### Main Invariant 3

```text
Message identity must prevent duplicate finalization.
```

## Additional Invariants

### Additional Invariant 1

```text
L1 release must only happen through a valid L2 -> L1 message.
```

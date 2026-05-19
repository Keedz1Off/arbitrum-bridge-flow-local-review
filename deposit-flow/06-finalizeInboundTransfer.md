# Function Review: finalizeInboundTransfer(...)

## Function Code

```solidity
// Paste the full finalizeInboundTransfer(...) function code here.
// Use the exact code from the contract version you are reviewing.

function finalizeInboundTransfer(
    // paste exact parameters here
) external payable {
    // paste exact function body here
}
```

---

## Function Explanation

`finalizeInboundTransfer(...)` finalizes the deposit on the destination chain.

For the deposit flow, this function is executed on L2 after the L1 -> L2 message arrives.

Main idea:

```text
Only an authentic bridge message should be able to finalize a deposit.
```

This function is critical because it turns decoded cross-chain message data into real L2 token credit.

---

## Important Logic Notes

### Authentication Check

If the function checks `msg.sender`, bridge messenger, or counterpart gateway, this is the auth boundary.

The function must make sure the call came from the expected bridge path.

Weak auth here can allow forged deposits.

---

### Address Aliasing

In Arbitrum, if an L1 contract sends a message to L2, the L2 side may see an aliased sender address.

```text
raw L1 gateway address != aliased L2 sender address
```

If this function checks the raw address when it should check the aliased address, sender validation may be wrong.

---

### Decoded Transfer Values

This function usually receives or decodes:

- token
- sender
- recipient
- amount
- extra data

These values must match what was encoded on L1.

---

### Mint / Release Step

After validation, this function may call mint or inbound escrow release logic.

That final token credit must use the verified amount and recipient.

---

## Invariants

- Only an authentic L1 -> L2 bridge message may finalize a deposit.
- The counterpart gateway must be the expected gateway.
- Address aliasing must be handled correctly when relevant.
- Decoded amount must match the amount backed by L1 escrow.
- Decoded recipient must match the intended L2 recipient.
- Decoded token must match the correct token mapping.
- The same message must not finalize twice.

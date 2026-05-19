# Function Review: outboundTransfer(...) / withdraw(...)

## Function Code

```solidity
// Paste the full outboundTransfer(...) or withdraw(...) function code here.
// Use the exact code from the contract version you are reviewing.

function withdraw(
    // paste exact parameters here
) external payable {
    // paste exact function body here
}
```

---

## Function Explanation

`outboundTransfer(...)` / `withdraw(...)` is the main L2 entry point for starting a withdrawal back to L1.

The user calls this function when they want to move value from L2 to L1.

At a high level, this function usually:

- receives withdrawal parameters from the user
- selects the correct token/gateway path
- burns or locks tokens on L2
- builds calldata for L1 finalization
- creates the outbound L2 -> L1 message

Main idea:

```text
The L1 release message must represent what actually happened on L2.
```

---

## Important Logic Notes

### User Inputs

Important values usually include:

- token
- recipient
- amount
- extra data

These values must not corrupt the withdrawal message.

---

### Token / Gateway Selection

The selected L2 token must map to the correct L1 token and gateway.

```text
L2 token -> correct L1 token / gateway
```

Wrong mapping may release the wrong asset on L1.

---

### Burn / Lock Step

The function should remove or lock the user's L2 token balance before L1 release can happen.

```text
burned / locked on L2 -> released on L1
```

If the message is created without a real burn/lock, L1 may release unbacked funds.

---

## Invariants

- L2 burned / locked amount must equal L1 released amount.
- The selected L2 token must map to the correct L1 token.
- The L1 recipient must match the intended recipient.
- Burn/lock must happen before L1 release is finalized.
- The amount encoded for L1 must match the real burned / locked amount.
- The outbound message must target the correct L1 gateway.
- The withdrawal message must not be finalized twice.

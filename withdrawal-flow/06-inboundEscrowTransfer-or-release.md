# Function Review: inboundEscrowTransfer(...) / release(...)

## Function Code

```solidity
// Paste the full inboundEscrowTransfer(...) or release(...) function code here.
// Use the exact code from the contract version you are reviewing.

function inboundEscrowTransfer(
    // paste exact parameters here
) internal {
    // paste exact function body here
}
```

---

## Function Explanation

`inboundEscrowTransfer(...)` / `release(...)` is the final token release step in the withdrawal flow.

After the withdrawal message is verified on L1, this function releases escrowed tokens to the L1 recipient.

Main idea:

```text
The final L1 release must match the verified L2 burn/lock.
```

This is where bridge accounting becomes an actual L1 token balance change.

---

## Important Logic Notes

### Authorized Caller

Release logic should only be callable by the trusted gateway/finalizer.

If unauthorized callers can trigger release, escrowed funds may be drained.

---

### Amount Used for Release

The released amount should come from verified withdrawal finalization data.

It should not be independently controlled by a user at this stage.

---

### Recipient

The recipient must match the L1 recipient encoded in the authenticated withdrawal message.

If the recipient can be changed here, funds can be redirected.

---

### Token Transfer Behavior

This step transfers real L1 tokens from escrow.

If the token behaves unexpectedly, the release may fail or create inconsistent accounting.

---

## Invariants

- Released amount on L1 must equal the verified burned / locked amount on L2.
- Only the authorized gateway/finalizer may release escrowed tokens.
- The recipient must match the finalized withdrawal message.
- The released token must be the expected L1 token.
- Escrow must not release more tokens than it holds.
- The same finalized message must not trigger release twice.

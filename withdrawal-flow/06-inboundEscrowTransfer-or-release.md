# Function Review: inboundEscrowTransfer(...) / release(...)

## Function Code

```solidity
function inboundEscrowTransfer(
    address _l1Token,
    address _dest,
    uint256 _amount
) internal virtual {
    // this method is virtual since different subclasses can handle escrow differently
    IERC20(_l1Token).safeTransfer(_dest, _amount);
}
```

---

## Function Explanation

`inboundEscrowTransfer(...)` / `release(...)` is the final token release step in the withdrawal flow.

After the withdrawal message is verified on L1, this function releases escrowed tokens to the L1 recipient.

Main idea:

```text
The final L1 release must match the verified L2 burn.
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

### Main Invariant 1

```text
Released amount on L1 must equal the verified burned amount on L2.
```

### Main Invariant 2

```text
Only the authorized gateway/finalizer may release escrowed tokens.
```

### Main Invariant 3

```text
The recipient must match the finalized withdrawal message.
```

## Additional Invariants

### Additional Invariant 1

```text
The released token must be the expected L1 token.
```

### Additional Invariant 2

```text
Escrow must not release more tokens than it holds.
```

### Additional Invariant 3

```text
The same finalized message must not trigger release twice.
```

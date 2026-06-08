# Function Review: burn(...) / lock(...)

## Function Code

```solidity
function outboundEscrowTransfer(
    address _l2Token,
    address _from,
    uint256 _amount
) internal virtual returns (uint256 amountBurnt) {
    // this method is virtual since different subclasses can handle escrow differently
    // user funds are escrowed on the gateway using this function
    // burns L2 tokens in order to release escrowed L1 tokens
    IArbToken(_l2Token).bridgeBurn(_from, _amount);
    // by default we assume that the amount we send to bridgeBurn is the amount burnt
    // this might not be the case for every token
    return _amount;
}
```

---

## Function Explanation

`burn(...)` / `lock(...)` is the L2 accounting step in the withdrawal flow.

This function removes or locks the user's L2 token balance before the bridge requests release on L1.

Main idea:

```text
L1 must release only what was actually burned or locked on L2.
```

If this step does not happen correctly, the bridge may release L1 escrow without proper L2 backing.

---

## Important Logic Notes

### Burn Logic

If the function burns tokens, it should reduce the user's L2 balance or token supply.

This creates the economic basis for releasing tokens on L1.

---

### Lock Logic

If the function locks tokens instead of burning, the locked amount must be tracked correctly.

The locked amount should match the amount encoded for L1 release.

---

### Amount Used Downstream

The amount used in the L1 withdrawal message should match the real burned or locked amount.

If burn/lock fails or moves a different amount, the withdrawal message must not proceed with the wrong value.

---

## Invariants

### Main Invariant 1

```text
Tokens must be burned or locked before L1 release.
```

### Main Invariant 2

```text
Burned / locked amount must equal L1 released amount.
```

### Main Invariant 3

```text
Failed burn/lock must stop the withdrawal flow.
```

## Additional Invariants

### Additional Invariant 1

```text
The token being burned/locked must be the intended L2 token.
```

### Additional Invariant 2

```text
The amount used downstream must match the real burned / locked amount.
```

### Additional Invariant 3

```text
L1 release must not happen without successful L2 burn/lock.
```

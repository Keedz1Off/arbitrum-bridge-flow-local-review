# Function Review: burn(...) / lock(...)

## Function Code

```solidity
// Paste the full burn(...) or lock(...) function code here.
// Use the exact code from the contract version you are reviewing.

function burn(
    // paste exact parameters here
) internal {
    // paste exact function body here
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

- Tokens must be burned or locked before L1 release.
- Burned / locked amount must equal L1 released amount.
- Failed burn/lock must stop the withdrawal flow.
- The token being burned/locked must be the intended L2 token.
- The amount used downstream must match the real burned / locked amount.
- L1 release must not happen without successful L2 burn/lock.

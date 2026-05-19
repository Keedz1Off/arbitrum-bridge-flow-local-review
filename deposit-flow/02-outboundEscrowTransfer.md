# Function Review: outboundEscrowTransfer(...)

## Function Code

```solidity
// Paste the full outboundEscrowTransfer(...) function code here.
// Use the exact code from the contract version you are reviewing.

function outboundEscrowTransfer(
    // paste exact parameters here
) internal returns (uint256) {
    // paste exact function body here
}
```

---

## Function Explanation

`outboundEscrowTransfer(...)` is the L1 escrow step in the deposit flow.

It is usually called inside `outboundTransfer(...)`.

This function is responsible for moving or locking the user's tokens on L1 before the bridge creates or finalizes the L2 credit.

At a high level, this function usually:

- receives the token address
- receives the sender address
- receives the requested amount
- transfers tokens from the user to the bridge/escrow
- returns or defines the amount that should be used for downstream accounting

The main security idea is:

```text
The bridge must credit L2 only for the amount that was actually received or escrowed on L1.
```

If this function does not correctly handle the token transfer, the entire deposit accounting can become wrong.

---

## Important Logic Notes

### Token Transfer / Escrow

If the function performs a token transfer using something like:

```solidity
transferFrom(...);
```

or:

```solidity
safeTransferFrom(...);
```

this is the core escrow operation.

This is where the bridge should actually receive the user's L1 tokens.

If this step fails or behaves unexpectedly, the bridge must not continue as if the deposit succeeded.

---

### Actual Received Amount

The most important point in this function is the difference between `amount` and `actualReceived`.

```text
amount = what the user requested to transfer
actualReceived = what the bridge actually received
```

For standard ERC20 tokens, these values usually match.

For fee-on-transfer tokens, they may differ.

Example:

```text
User sends: 100
Bridge receives: 98
Fee taken by token: 2
```

The safest accounting pattern is:

```text
actualReceived = balanceAfter - balanceBefore
```

If the bridge later credits L2 using `amount` instead of `actualReceived`, L2 may receive more value than L1 escrow actually holds.

---

### Non-Standard Token Behavior

Some tokens do not behave like normal ERC20 tokens.

Examples:

- fee-on-transfer tokens
- false-return tokens
- rebasing tokens
- pausable or blacklistable tokens
- malicious tokens with callbacks/hooks

This matters because the bridge may assume that a token transfer succeeded and moved exactly `amount`, but the real balance change may be different.

---

### Return Value

If the function returns an amount, that returned value is important.

It may be used later for:

- calldata construction
- L2 mint/release amount
- accounting updates

The returned value should represent the real escrowed/received amount, not only the user-requested amount.

---

## Invariants

- Tokens must actually move from the user to the L1 bridge/escrow.
- Failed token transfer must stop the deposit flow.
- L1 escrowed amount must equal L2 minted / released amount.
- The amount used downstream should match the real received / escrowed amount.
- For fee-on-transfer or non-standard tokens, accounting should not blindly trust user-supplied `amount`.
- The token being escrowed must be the intended L1 token.
- L2 credit must not happen without successful L1 escrow.

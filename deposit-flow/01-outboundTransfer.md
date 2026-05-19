# Function Review: outboundTransfer(...)

## Function Code

```solidity
// Paste the full outboundTransfer(...) function code here.
// Use the exact code from the contract version you are reviewing.

function outboundTransfer(
    // paste exact parameters here
) external payable returns (bytes memory) {
    // paste exact function body here
}
```

---

## Function Explanation

`outboundTransfer(...)` is the main L1 entry point for starting a deposit into Arbitrum.

The user calls this function to bridge tokens from L1 to L2.

At a high level, this function usually:

- receives deposit parameters from the user
- selects the correct token/gateway path
- moves tokens into L1 escrow
- builds calldata for the L2 message
- creates the retryable ticket that will later execute on L2

The main security idea is:

```text
The L2 message must represent what actually happened on L1.
```

If the function creates an L2 message using wrong or unverified values, the destination chain may credit tokens that are not correctly backed by source-chain escrow.

---

## Important Logic Notes

### Token / Gateway Selection

If the function selects or resolves an L2 token/gateway, this part defines which token or gateway will be used on the destination chain.

This matters because the bridge must preserve token identity across chains.

```text
L1 token -> correct L2 token / gateway
```

If this mapping is wrong, the bridge may lock one asset on L1 but credit another asset on L2.

---

### Escrow Step

If the function calls something like:

```solidity
outboundEscrowTransfer(...);
```

this is the accounting boundary.

This is where the bridge should actually receive or lock the user's L1 tokens.

Important distinction:

```text
amount = what the user requested to send
actualReceived = what the bridge actually received
```

For standard ERC20 tokens, these values usually match.

For fee-on-transfer or non-standard tokens, they may differ.

The safest accounting idea is:

```text
actualReceived = balanceAfter - balanceBefore
```

If the bridge receives less than the amount later encoded for L2, the bridge can credit unbacked value on L2.

---

### Calldata Construction

If the function builds calldata using something like:

```solidity
getOutboundCalldata(...);
```

this is the message construction boundary.

This step decides which values will be sent to L2.

Important values:

- token
- recipient
- amount
- extra data

Calldata does not prove correctness by itself.

It only stores the values that the contract decided to encode.

So the important point is:

```text
The values must be correct before they are encoded.
```

---

### Retryable Ticket Creation

If the function calls something like:

```solidity
createRetryableTicket(...);
```

this is the L1 -> L2 message boundary.

This step sends the prepared calldata to Arbitrum for L2 execution.

The retryable ticket should target the correct L2 gateway and carry the correct calldata.

A retryable ticket can be valid at the messaging layer but still contain wrong bridge data.

---

## Invariants

- L1 escrowed amount must equal L2 minted / released amount.
- The selected L1 token must be the intended token.
- The selected L1 token must map to the correct L2 token.
- The L2 recipient must match the intended recipient.
- Tokens must be escrowed / locked on L1 before L2 credit is finalized.
- The amount encoded for L2 must match the real escrowed / received amount.
- The calldata must match what the L2 finalize function expects.
- The retryable ticket must target the correct L2 gateway.
- The deposit message must not be finalized twice.

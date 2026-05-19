# Function Review: inboundEscrowTransfer(...) / mint(...)

## Function Code

```solidity
function inboundEscrowTransfer(
    address _l2Address,
    address _dest,
    uint256 _amount
) internal virtual {
    // this method is virtual since different subclasses can handle escrow differently
    IArbToken(_l2Address).bridgeMint(_dest, _amount);
}
```

---

## Function Explanation

`inboundEscrowTransfer(...)` / `mint(...)` is the final token credit step in the deposit flow.

After the bridge message is finalized, this function releases or mints tokens to the L2 recipient.

Main idea:

```text
The final token credit must match the verified bridge message.
```

This is where bridge accounting becomes an actual user token balance.

---

## Important Logic Notes

### Authorized Caller

Mint or release logic should only be callable by the trusted gateway/finalizer.

If this function is callable by an unauthorized account, tokens may be minted or released without a real bridge message.

---

### Amount Used for Credit

The credited amount should come from verified finalization data.

It should not be independently controlled by a user at this stage.

---

### Recipient

The recipient must be the same recipient that was encoded in the authenticated bridge message.

If the recipient can be changed here, funds can be redirected.

---

### Token Behavior

If this step transfers tokens instead of minting, token behavior may matter.

If this step mints representation tokens, minter permissions and supply accounting matter.

---

## Invariants

- Minted / released amount on L2 must equal the verified L1 escrowed amount.
- Only the authorized gateway/finalizer may mint or release tokens.
- The recipient must match the finalized bridge message.
- The credited token must be the expected L2 token.
- L2 token supply must remain backed by L1 escrow.
- The same finalized message must not trigger mint/release twice.

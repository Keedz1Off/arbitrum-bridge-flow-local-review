
# Function Review: finalizeInboundTransfer(...)

## 1. Function Code

```solidity
// Paste the exact finalizeInboundTransfer(...) source code here.
//
// Example structure:
//
// function finalizeInboundTransfer(
//     address _token,
//     address _from,
//     address _to,
//     uint256 _amount,
//     bytes calldata _data
// ) external payable {
//     ...
// }
```

---

## 2. Function Purpose

`finalizeInboundTransfer(...)` finalizes the deposit on the destination chain.

For deposit flow, this function is executed on L2 after the L1 -> L2 message is
delivered.

In simple terms:

```text
This function receives the bridge message and credits the user on L2.
```

It is critical because this is where cross-chain message data becomes actual L2
token credit.

---

## 3. Critical Parameters

- `_token`: token being finalized
- `_from`: original sender / depositor
- `_to`: recipient on destination chain
- `_amount`: amount to mint or release
- `_data`: additional encoded data
- `msg.sender`: immediate caller
- original cross-chain sender / counterpart gateway

---

## 4. Trusted / Untrusted Input

Trusted only if verified:

- counterpart gateway
- bridge messenger / inbox sender
- decoded calldata from authentic message

Untrusted:

- direct external calls
- spoofed messages
- unvalidated `_data`
- decoded values without sender verification
- raw `msg.sender` if aliasing is ignored

Important audit idea:

```text
Finalization must only happen through an authentic bridge message.
```

---

## 5. Called After

Typical deposit execution path:

```text
createRetryableTicket(...)
└── finalizeInboundTransfer(...) on L2
    └── inboundEscrowTransfer(...) / mint(...)
```

Security meaning:

```text
This is the destination-chain finalization boundary.
```

---

## 6. Invariants

- Only the authentic L1 counterpart gateway may trigger finalization.
- The L2 credited amount must equal the valid L1 escrowed amount.
- The recipient must match the encoded recipient.
- The token must match the expected token mapping.
- The same message must not finalize twice.
- Address aliasing must be handled correctly if the sender is an L1 contract.

---

## 7. Authentication Boundary

The key question:

```text
Who is allowed to finalize the transfer?
```

A strong check should verify more than only `msg.sender`.

It should verify:

- bridge messenger / system caller
- original L1 sender
- expected counterpart gateway
- correct bridge path
- replay protection

For Arbitrum L1 contract messages, sender aliasing may apply:

```text
Expected L1 gateway address -> expected aliased L2 sender
```

---

## 8. Security Risks

### Risk 1: Direct call without proper auth

If anyone can call `finalizeInboundTransfer(...)`, an attacker may mint or release
tokens without a real deposit.

Impact:

```text
Forged deposit / unbacked token credit.
```

### Risk 2: Wrong counterpart gateway

If the function does not verify the expected counterpart gateway, a spoofed
message may finalize a fake transfer.

Impact:

```text
Cross-chain auth bypass.
```

### Risk 3: Address aliasing mistake

If the function checks the raw L1 gateway address instead of the expected aliased
address, auth may be wrong.

Impact:

```text
Broken Arbitrum sender validation.
```

### Risk 4: Amount corruption

If decoded `_amount` does not match the amount escrowed on L1, L2 may credit too
much or too little.

Impact:

```text
Broken token conservation.
```

### Risk 5: Recipient substitution

If `_to` can be changed or decoded incorrectly, funds may be credited to the
wrong recipient.

Impact:

```text
Loss or redirection of user funds.
```

### Risk 6: Replay

If the same message can be finalized twice, L2 may mint/release twice for one L1
deposit.

Impact:

```text
Double credit / bridge insolvency.
```

---

## 9. Audit Questions

- Can `finalizeInboundTransfer(...)` be called directly?
- Does it verify the bridge messenger?
- Does it verify the original L1 sender?
- Does it verify the expected counterpart gateway?
- Does it handle Arbitrum address aliasing correctly?
- Can `_amount`, `_token`, or `_to` be spoofed?
- Does decoded calldata match the L1 encoding format?
- Is the L2 token mapping checked?
- Can the same message be executed twice?
- Does finalization call mint or escrow release safely?
- Are there external calls after state changes?

---

## 10. Audit Conclusion

`finalizeInboundTransfer(...)` is the destination-chain finalization boundary.

Most important invariant:

```text
Only an authentic bridge message may mint or release tokens on L2.
```

Main security question:

```text
Is this finalization backed by a real L1 deposit?
```

Main risk:

```text
A spoofed, replayed, or incorrectly decoded message credits value on L2 without valid L1 backing.
```

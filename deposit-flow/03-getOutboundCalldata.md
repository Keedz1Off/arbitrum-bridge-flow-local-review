# Function Review: getOutboundCalldata(...)

## 1. Function Code

```solidity
// Paste the exact getOutboundCalldata(...) source code here.
//
// Example structure:
//
// function getOutboundCalldata(
//     address _l1Token,
//     address _from,
//     address _to,
//     uint256 _amount,
//     bytes memory _data
// ) public view returns (bytes memory) {
//     ...
// }
```

Note:

```text
The exact function body should be copied from the reviewed contract version.
Different gateway implementations may build outbound calldata differently.
```

---

## 2. Function Purpose

`getOutboundCalldata(...)` builds the calldata / payload for the message that will
be executed on L2.

This function is usually called inside `outboundTransfer(...)` after the
escrow/lock step.

In simple terms:

```text
This function packages the deposit parameters into a message for L2.
```

It is critical because values such as `token`, `from`, `to`, and `amount` are
turned into encoded calldata here.

If the wrong `amount`, `token`, or `recipient` is encoded, L2 may finalize the
deposit incorrectly.

---

## 3. Critical Parameters

Common critical parameters:

- `l1Token` / `_l1Token`
- `from` / `_from`
- `to` / `_to`
- `amount` / `_amount`
- `data` / `_data`

Parameter meaning:

- `l1Token`: L1 token involved in the deposit
- `from`: user or deposit initiator
- `to`: L2 recipient
- `amount`: amount that will be encoded for L2
- `data`: additional data that may affect downstream execution

---

## 4. Trusted / Untrusted Input

Trusted or semi-trusted:

- validated token mapping
- configured gateway addresses
- internally computed amount, if already verified
- internally selected L2 target

Untrusted:

- user-supplied `amount`, if not verified
- user-supplied `to`
- user-supplied `_data`
- token address, if not validated
- any value that enters calldata without validation

Important audit idea:

```text
calldata does not prove that the parameters are correct.
```

The contract must first choose and validate the correct values, and only then
encode them.

---

## 5. Called Inside

Typical call structure:

```text
outboundTransfer(...)
└── getOutboundCalldata(...)
```

`getOutboundCalldata(...)` is usually an internal/message construction step
inside the deposit flow.

Security meaning:

```text
This is the calldata construction boundary.
```

At this step, the bridge decides which values will be passed to L2.

---

## 6. Invariants

Main calldata invariant:

```text
Encoded calldata must represent the real L1 deposit.
```

Function-level invariants:

- Encoded `amount` must match the escrowed / actual received amount.
- Encoded `to` must be the intended L2 recipient.
- Encoded `l1Token` must match the expected token mapping.
- L1 encode format must match L2 decode format.
- `_data` must not override critical parameters in an unsafe way.
- Calldata must target the correct L2 finalize logic.

---

## 7. Calldata Consistency

Bridge flow usually looks like this:

```text
values selected on L1
-> abi.encode(...) / abi.encodeWithSelector(...)
-> retryable ticket
-> decode on L2
-> finalizeInboundTransfer(...)
```

Main question:

```text
Are the encoded values the same values that should actually reach L2?
```

Values to trace:

```text
input amount
-> escrowed / actual received amount
-> encoded amount
-> decoded amount on L2
-> minted / released amount
```

If `getOutboundCalldata(...)` encodes the wrong amount, downstream logic may be
formally correct but still execute the wrong transfer.

Example risk:

```text
L1 actually received: 98
getOutboundCalldata encodes: 100
L2 finalizes: 100
```

Broken invariant:

```text
Encoded amount != actual L1 escrowed amount
```

---

## 8. Security Risks

### Risk 1: Wrong amount encoded

If the function encodes user-supplied `amount` instead of actual received /
escrowed amount, L2 may credit more value than L1 actually received.

Impact:

```text
Unbacked L2 tokens / broken bridge accounting.
```

### Risk 2: Recipient substitution

If `to` can be changed through `_data` or incorrect decode logic, L2 may credit
tokens to the wrong recipient.

Impact:

```text
Funds credited to the wrong address.
```

### Risk 3: Wrong token encoded

If the wrong `l1Token` or mapped token enters calldata, L2 may use an incorrect
token mapping.

Impact:

```text
Token identity corruption.
```

### Risk 4: Encode/decode mismatch

If the L1 encode format does not match the L2 decode format, L2 may interpret the
data incorrectly.

Impact:

```text
Amount / recipient / token corruption during finalization.
```

### Risk 5: Unsafe extra data

If `_data` contains nested parameters and they are used without strict
validation, the user may influence L2 execution in a non-obvious way.

Impact:

```text
Unexpected calldata behavior or parameter override.
```

---

## 9. Audit Questions

- Who calls `getOutboundCalldata(...)`?
- Is the function `public`, `external`, `internal`, or `view`?
- Which exact values are encoded?
- Does encoded `amount` come from user input or actual received amount?
- Can `_data` modify `amount`, `token`, or `recipient`?
- Does the L1 encode format match the L2 decode format?
- Which selector is used in calldata?
- Which L2 function will be called by this calldata?
- Is token mapping checked before encoding?
- Can the recipient be substituted?
- Can calldata be reused or replayed?
- Are there downstream checks on L2 after decode?

---

## 10. Audit Conclusion

`getOutboundCalldata(...)` is the message construction boundary of the deposit
flow.

Main security question:

```text
What exactly does the bridge send to L2?
```

Most important invariant:

```text
Encoded calldata must match the real L1 deposit.
```

If this function encodes the wrong `amount`, `token`, or `recipient`, L2 may
correctly execute the message but still produce an economically incorrect result.

Main risk:

```text
Calldata represents user input instead of verified source-chain reality.
```

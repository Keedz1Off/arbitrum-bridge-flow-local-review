# Function Review: outboundEscrowTransfer(...)

## 1. Function Code

```solidity
// Paste the exact outboundEscrowTransfer(...) source code here.
//
// Example structure:
//
// function outboundEscrowTransfer(
//     address _l1Token,
//     address _from,
//     uint256 _amount
// ) internal returns (uint256) {
//     ...
// }
```

Note:

```text
The exact function body should be copied from the reviewed contract version.
Different gateway implementations may implement escrow logic differently.
```

---

## 2. Function Purpose

`outboundEscrowTransfer(...)` is the deposit-side escrow step.

It is usually called inside `outboundTransfer(...)`.

Its purpose is to move or lock the user's L1 tokens into the bridge escrow before
the L2 deposit message is finalized.

In simple terms:

```text
This function is where the bridge should actually receive the user's tokens on L1.
```

This function is critical because it connects the user's requested deposit amount
with the bridge's real source-chain token balance.

---

## 3. Critical Parameters

Common critical parameters:

- `l1Token` / `_l1Token`
- `from` / `_from`
- `amount` / `_amount`

Parameter meaning:

- `l1Token`: token being escrowed on L1
- `from`: address sending tokens into escrow
- `amount`: requested amount to move from the user into the bridge

---

## 4. Trusted / Untrusted Input

Trusted or semi-trusted:

- gateway escrow address
- validated token mapping
- internal caller, if function is internal
- configured token address, if validated before call

Untrusted:

- user-supplied `amount`
- token contract behavior
- ERC20 return behavior
- fee-on-transfer logic
- rebasing or balance-changing behavior
- malicious token callbacks/hooks

Important audit idea:

```text
The bridge should care about how many tokens it actually received, not only how many tokens the user requested to send.
```

---

## 5. Called Inside

Typical call structure:

```text
outboundTransfer(...)
└── outboundEscrowTransfer(...)
```

`outboundEscrowTransfer(...)` is not usually the full deposit flow by itself.

It is an internal accounting step inside the larger `outboundTransfer(...)`
deposit process.

Security meaning:

```text
This is the accounting boundary of the deposit flow.
```

If this step is wrong, every downstream step can inherit the wrong amount.

---

## 6. Invariants

Main escrow invariant:

```text
Amount actually received by L1 escrow = amount credited on L2
```

Function-level invariants:

- Tokens must move from the user to the bridge escrow.
- Escrow must happen before L2 mint/release.
- The bridge must not credit more than it actually received.
- For non-standard tokens, accounting should use actual received amount.
- Failed token transfer must revert the deposit.
- The token address must be the expected L1 token.

---

## 7. Amount Consistency

This function is the most important place to check `actualReceived`.

Correct accounting pattern:

```text
balanceBefore = token.balanceOf(address(this))
transferFrom(user, address(this), amount)
balanceAfter = token.balanceOf(address(this))

actualReceived = balanceAfter - balanceBefore
```

Why this matters:

```text
amount = what the user asked to transfer
actualReceived = what the bridge actually received
```

If the bridge uses `amount` but receives less, accounting can break.

Example:

```text
User requested deposit: 100
Token fee: 2
Bridge actually received: 98
L2 credited amount: 100
```

Broken invariant:

```text
L1 escrowed amount < L2 minted / released amount
```

---

## 8. Security Risks

### Risk 1: Fee-on-transfer amount mismatch

A fee-on-transfer token may deduct a fee during transfer.

Example:

```text
User sends 100
Bridge receives 98
2 tokens are taken as fee
```

If the bridge still credits 100 on L2, the bridge creates more L2 value than it
received on L1.

Impact:

```text
Unbacked L2 tokens / accounting insolvency.
```

### Risk 2: False-return token

Some ERC20 tokens return `false` instead of reverting when transfer fails.

If the bridge does not use safe transfer logic, it may continue as if the
transfer succeeded.

Impact:

```text
L2 credit without real L1 escrow.
```

### Risk 3: Non-standard token behavior

Some tokens may rebase, take fees, blacklist, pause, or change balances in
unexpected ways.

Impact:

```text
Escrow accounting may not match bridge assumptions.
```

### Risk 4: Reentrancy through token callbacks

A malicious token may try to reenter bridge logic during transfer.

Impact:

```text
Unexpected state changes or repeated flow execution.
```

### Risk 5: Wrong token address

If the escrow function receives or uses an incorrect token address, the bridge
may escrow one asset but credit another.

Impact:

```text
Token mapping/accounting corruption.
```

---

## 9. Audit Questions

- Is `outboundEscrowTransfer(...)` internal or externally callable?
- Who can trigger this function?
- Is the token address validated before transfer?
- Does the function use `safeTransferFrom` or equivalent?
- Does it check the actual received amount?
- Does it calculate `actualReceived = balanceAfter - balanceBefore`?
- Does downstream calldata use `amount` or `actualReceived`?
- What happens if the token charges a transfer fee?
- What happens if the token returns `false`?
- What happens if the token is rebasing?
- Can a malicious token reenter bridge logic?
- Does the deposit revert if escrow transfer fails?
- Can L2 credit happen without successful L1 escrow?

---

## 10. Audit Conclusion

`outboundEscrowTransfer(...)` is the deposit accounting boundary.

The main security question is:

```text
How many tokens did the bridge actually receive?
```

Most important invariant:

```text
Actual L1 escrowed amount = L2 credited amount
```

If this function trusts the requested `amount` instead of measuring or safely
verifying the actual received amount, the bridge may mint or release unbacked
tokens on L2.

Main risk:

```text
User-supplied amount is treated as real escrowed value.
```


# Break Think Analysis

This folder is for manual Break Think analysis.

Break Think means:

```text
Invariant -> Consequence
```

## Main Deposit Functions

```text
outboundTransfer(...)
outboundEscrowTransfer(...)
createRetryableTicket(...)
AbsInbox._createRetryableTicket(...)
finalizeInboundTransfer(...)
```

## Main Withdrawal Functions

```text
outboundTransfer(...) / withdraw(...)
burn(...)
finalizeInboundTransfer(...) / finalizeWithdrawal(...)
```

## Format

```text


Invariant:

Consequences:
```
## Important 

Here is most important functions!!!

## PoC Testing Note

The `exploit-labs/` folder contains simplified PoC-style examples for bridge security issues.

There are two useful ways to write PoC tests:

```text
Vulnerable case:
attack succeeds -> assert the bad state
```

```text
Protected / fixed case:
attack must revert -> use vm.expectRevert(...)
```

Important:

```text
PASS with vm.expectRevert = protection worked.
PASS without revert + bad state = vulnerability was proven.
```






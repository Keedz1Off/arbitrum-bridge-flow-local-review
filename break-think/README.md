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
finalizeInboundTransfer(...)
```

## Main Withdrawal Functions

```text
outboundTransfer(...) / withdraw(...)
burn(...) / lock(...)
finalizeInboundTransfer(...) / finalizeWithdrawal(...)
```

## Format

```text
Function:

Invariant:

Consequences:
```

## Goal

For each important function, choose the main invariants and explain what can happen if they break.

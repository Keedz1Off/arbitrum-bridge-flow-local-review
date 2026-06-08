# Function Review: finalizeInboundTransfer(...) / finalizeWithdrawal(...)

## Function Code

```solidity
function finalizeInboundTransfer(
    address _token,
    address _from,
    address _to,
    uint256 _amount,
    bytes calldata _data
) public payable virtual override onlyCounterpartGateway {
    // this function is marked as virtual so superclasses can override it to add modifiers
    (uint256 exitNum, bytes memory callHookData) = GatewayMessageHandler.parseToL1GatewayMsg(
        _data
    );

    if (callHookData.length != 0) {
        // callHookData should always be 0 since inboundEscrowAndCall is disabled
        callHookData = bytes("");
    }

    // we ignore the returned data since the callHook feature is now disabled
    (_to, ) = getExternalCall(exitNum, _to, callHookData);
    inboundEscrowTransfer(_token, _to, _amount);

    emit WithdrawalFinalized(_token, _from, _to, exitNum, _amount);
}
```

---

## Function Explanation

`finalizeInboundTransfer(...)` / `finalizeWithdrawal(...)` finalizes the withdrawal on L1.

This function is executed after the L2 -> L1 message is proven or accepted by the bridge system.

Main idea:

```text
Only an authentic L2 withdrawal message should release funds on L1.
```

This is where message data becomes real L1 token release.

---

## Important Logic Notes

### Authentication Check

The function must verify that the message came from the expected L2 gateway through the correct bridge path.

Weak authentication here can allow forged withdrawals.

---

### Decoded Transfer Values

The function usually receives or decodes:

- token
- sender
- recipient
- amount
- extra data

These values must match the message created on L2.

---

### Release Step

After verification, this function releases escrowed L1 tokens to the recipient.

The release amount should match the verified burned / locked amount from L2.

---

## Invariants

### Main Invariant 1

```text
Only an authentic L2 -> L1 bridge message may finalize a withdrawal.
```

### Main Invariant 2

```text
The counterpart gateway must be the expected gateway.
```

### Main Invariant 3

```text
Decoded amount must match the amount burned / locked on L2.
```

## Additional Invariants

### Additional Invariant 1

```text
Decoded recipient must match the intended L1 recipient.
```

### Additional Invariant 2

```text
Decoded token must match the correct L1 token.
```

### Additional Invariant 3

```text
The same withdrawal message must not finalize twice.
```

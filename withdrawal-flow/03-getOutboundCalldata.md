# Function Review: getOutboundCalldata(...)

## Function Code

```solidity
function getOutboundCalldata(
    address _token,
    address _from,
    address _to,
    uint256 _amount,
    bytes memory _data
) public view override returns (bytes memory outboundCalldata) {
    outboundCalldata = abi.encodeWithSelector(
        ITokenGateway.finalizeInboundTransfer.selector,
        _token,
        _from,
        _to,
        _amount,
        GatewayMessageHandler.encodeFromL2GatewayMsg(exitNum, _data)
    );

    return outboundCalldata;
}
```

---

## Function Explanation

`getOutboundCalldata(...)` builds the calldata / payload for the L2 -> L1 withdrawal message.

This calldata will later be decoded on L1 during withdrawal finalization.

Main idea:

```text
The L1 finalization calldata must represent the real L2 withdrawal.
```

---

## Important Logic Notes

### Encoding Step

If the function uses `abi.encode(...)` or `abi.encodeWithSelector(...)`, this is where withdrawal values become message data.

Important values usually include:

- L1 token
- L2 token
- sender
- recipient
- amount
- extra data

---

### Amount in Calldata

The encoded amount should match the amount actually burned or locked on L2.

If the encoded amount is larger than the burned/locked amount, L1 may release too much.

---

### Encode / Decode Match

The L2 encode format must match the L1 decode format.

```text
L2 encode format = L1 decode format
```

If the formats differ, L1 may decode the wrong amount, token, or recipient.

---

## Invariants

### Main Invariant 1

```text
Encoded calldata must represent the real L2 withdrawal.
```

### Main Invariant 2

```text
Encoded amount must match the burned / locked amount.
```

### Main Invariant 3

```text
Encoded recipient must match the intended L1 recipient.
```

## Additional Invariants

### Additional Invariant 1

```text
Encoded token must match the correct L1 token mapping.
```

### Additional Invariant 2

```text
L2 encoding must match L1 decoding.
```

### Additional Invariant 3

```text
Extra data must not override critical values in an unsafe way.
```

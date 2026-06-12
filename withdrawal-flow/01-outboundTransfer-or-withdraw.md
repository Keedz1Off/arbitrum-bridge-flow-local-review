# Function Review: outboundTransfer(...) / withdraw(...)

## Function Code

```solidity
function outboundTransfer(
    address _l1Token,
    address _to,
    uint256 _amount,
    uint256, /* _maxGas */
    uint256, /* _gasPriceBid */
    bytes calldata _data
) public payable virtual override returns (bytes memory res) {
    // This function is set as public and virtual so that subclasses can override
    // it and add custom validation for callers (ie only whitelisted users)

    // the function is marked as payable to conform to the inheritance setup
    // this particular code path shouldn't have a msg.value > 0
    // TODO: remove this invariant for execution markets
    require(msg.value == 0, "NO_VALUE");

    address _from;
    bytes memory _extraData;
    {
        if (isRouter(msg.sender)) {
            (_from, _extraData) = GatewayMessageHandler.parseFromRouterToGateway(_data);
        } else {
            _from = msg.sender;
            _extraData = _data;
        }
    }
    // the inboundEscrowAndCall functionality has been disabled, so no data is allowed
    require(_extraData.length == 0, "EXTRA_DATA_DISABLED");

    uint256 id;
    {
        address l2Token = calculateL2TokenAddress(_l1Token);
        require(l2Token.isContract(), "TOKEN_NOT_DEPLOYED");
        require(_isValidTokenAddress(_l1Token, l2Token), "NOT_EXPECTED_L1_TOKEN");

        _amount = outboundEscrowTransfer(l2Token, _from, _amount);
        id = triggerWithdrawal(_l1Token, _from, _to, _amount, _extraData);
    }
    return abi.encode(id);
}
```

---

## Function Explanation

`outboundTransfer(...)` / `withdraw(...)` is the main L2 entry point for starting a withdrawal back to L1.

The user calls this function when they want to move value from L2 to L1.

At a high level, this function usually:

- receives withdrawal parameters from the user
- selects the correct token/gateway path
- burns tokens on L2
- builds calldata for L1 finalization
- creates the outbound L2 -> L1 message

Main idea:

```text
The L1 release message must represent what actually happened on L2.
```

---

## Important Logic Notes

### User Inputs

Important values usually include:

- token
- recipient
- amount
- extra data

These values must not corrupt the withdrawal message.

---

### Token / Gateway Selection

The selected L2 token must map to the correct L1 token and gateway.

```text
L2 token -> correct L1 token / gateway
```

Wrong mapping may release the wrong asset on L1.

---

### Burn Step

The function should burn the user's L2 token balance before L1 release can happen.

```text
burned on L2 -> released on L1
```

If the message is created without a real burn, L1 may release unbacked funds.

---

## Invariants

### Main Invariant 1

```text
L2 burned amount must equal L1 released amount.
```

### Main Invariant 2

```text
The selected L2 token must map to the correct L1 token.
```

### Main Invariant 3

```text
The L1 recipient must match the intended recipient.
```

## Additional Invariants

### Additional Invariant 1

```text
burn must happen before L1 release is finalized.
```

### Additional Invariant 2

```text
The amount encoded for L1 must match the real burned amount.
```

### Additional Invariant 3

```text
The outbound message must target the correct L1 gateway.
```

### Additional Invariant 4

```text
The withdrawal message must not be finalized twice.
```

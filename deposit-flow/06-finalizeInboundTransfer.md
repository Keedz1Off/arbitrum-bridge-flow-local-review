# Function Review: finalizeInboundTransfer(...)

## Function Code

```solidity
function finalizeInboundTransfer(
    address _token,
    address _from,
    address _to,
    uint256 _amount,
    bytes calldata _data
) external payable override onlyCounterpartGateway {
    (bytes memory gatewayData, bytes memory callHookData) = GatewayMessageHandler
        .parseFromL1GatewayMsg(_data);

    if (callHookData.length != 0) {
        // callHookData should always be 0 since inboundEscrowAndCall is disabled
        callHookData = bytes("");
    }

    address expectedAddress = calculateL2TokenAddress(_token);

    if (!expectedAddress.isContract()) {
        bool shouldHalt = handleNoContract(
            _token,
            expectedAddress,
            _from,
            _to,
            _amount,
            gatewayData
        );
        if (shouldHalt) return;
    }

    // validate if L1 address supplied matches that of the expected L2 address
    bool shouldWithdraw = !_isValidTokenAddress(_token, expectedAddress);
    if (shouldWithdraw) {
        // we don't need the return value from triggerWithdrawal since this is forcing
        // a withdrawal back to the L1 instead of composing with a L2 dapp
        triggerWithdrawal(_token, address(this), _from, _amount, "");
        return;
    }

    inboundEscrowTransfer(expectedAddress, _to, _amount);
    emit DepositFinalized(_token, _from, _to, _amount);

    return;
}
```

---

## Function Explanation

`finalizeInboundTransfer(...)` finalizes the deposit on the destination chain.

For the deposit flow, this function is executed on L2 after the L1 -> L2 message arrives.

Main idea:

```text
Only an authentic bridge message should be able to finalize a deposit.
```

This function is critical because it turns decoded cross-chain message data into real L2 token credit.

---

## Important Logic Notes

### Authentication Check

If the function checks `msg.sender`, bridge messenger, or counterpart gateway, this is the auth boundary.

The function must make sure the call came from the expected bridge path.

Weak auth here can allow forged deposits.

---

### Address Aliasing

In Arbitrum, if an L1 contract sends a message to L2, the L2 side may see an aliased sender address.

```text
raw L1 gateway address != aliased L2 sender address
```

If this function checks the raw address when it should check the aliased address, sender validation may be wrong.

---

### Decoded Transfer Values

This function usually receives or decodes:

- token
- sender
- recipient
- amount
- extra data

These values must match what was encoded on L1.

---

### Mint / Release Step

After validation, this function may call mint or inbound escrow release logic.

That final token credit must use the verified amount and recipient.

---

## Invariants

- Only an authentic L1 -> L2 bridge message may finalize a deposit.
- The counterpart gateway must be the expected gateway.
- Address aliasing must be handled correctly when relevant.
- Decoded amount must match the amount backed by L1 escrow.
- Decoded recipient must match the intended L2 recipient.
- Decoded token must match the correct token mapping.
- The same message must not finalize twice.

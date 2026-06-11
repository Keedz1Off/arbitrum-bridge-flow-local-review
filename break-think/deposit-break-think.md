# outboundTransfer()

   ## Invariants

1. L1 escrowed amount must equal L2 minted. 1,5. Or (actualReceived = balanceAfter - balanceBefore)

2. The selected L1 token must be the intended token.

   ## Concenquences
1. This may lead to a diffrent amount mint, this usually happens when encoded data (message). 1,5. This may lead to unbacked tokens.

2. This may lead to minting another token instead of the inteded token.


 # outboundEscrowTransfer()

  ## Invariants

1. Tokens must actually move from the user to the L1 bridge/escrow.

2. Failed token transfer must stop the deposit flow.

3. L1 escrowed amount must equal L2 minted. 1,5. Or (actualReceived = balanceAfter - balanceBefore).

 
  ## Concenquences
1. The token locking happens, but the message is not created.

2. The Funds stucks forever.

3. This may lead to a diffrent amount mint, this usually happens when encoded data (message). 1,5. This may lead to unbacked tokens. 

# createRetrybleTicket()

## Invariant

1. Retryable ticket must target the correct L2 gateway.

2. Retryable calldata must match the verified L1 deposit.

3. Retryable execution budget must be sufficient for finalization.

## Concenquences

1. This may lead to fund loss, because the L2 user did not get the fund.

2. An attacker changes the data that is may lead to minting the wrong amount on L2 (finalization)

3. This may lead to fund stuck (delayed finalization)

# AbsInbox-createRetrybleTicket()

## Invariants

The retryable ticket must preserve the intended L2 target.

The retryable ticket must preserve the intended calldata.

L2 sender assumptions must match Arbitrum aliasing rules.

## Concenquences

1. This may lead to minting on the wrong L2 bridge.

2. An attacker changes calldata (_to, _amount) which is may lead to wrong excution.

3. Without aliasing, L2 may see the wrong sender for the message.


# finalizeInboundTransfer(...)

 ## Invariants

1. Only an authentic L1 -> L2 bridge message may finalize a deposit.

2. The counterpart gateway must be the expected gateway.

3. Address aliasing must be handled correctly when relevant.


 ## Concenquences

1. Fake message can mint tokens on L2.

2. Wrong gateway can execute the wrong bridge message.

3. Without aliasing, L2 may see the wrong sender for the message.


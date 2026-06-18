# outboundTransfer()

## Invariant
1. L2 burned amount must equal L1 released amount.

2. The selected L2 token must map to the correct L1 token.

3. The L1 recipient must match the intended recipient.

## Concenquences

 1. The Bridge releases more or less tokens than the actually burned tokens.

 2. The token on L1 does not match the token on L2, that is user receives the wrong token.

 3. The tokens may be lost.



# burn()

## Invariants

1. Tokens must be burned  before L1 release.

2. Burned  amount must equal L1 released amount.

3. Failed burn  must stop the withdrawal flow


## Concenauences
1. This may lead to releasing without burning L2 tokens (message  --->  release or unlock)

2. The Bridge releases more or less tokens than the actually burned tokens,so an user receives diffrent amount.

3. This may lead to releasing without burning L2 tokens (message  --->  release or unlock)



# finalizeInboundTransfer()


## Invariants

1. Only an authentic L1 -> L2 bridge message may finalize a deposit.

2. The counterpart gateway must be the expected gateway.

3. Released amount must equal burned amount



## Concenauences

1. Anyone can call this function and bypass the  burning process.

2. This function may be called by a spoofed bridge.

3.. The Bridge releases more or less tokens than the actually burned tokens,so an user receives diffrent amount. 

done







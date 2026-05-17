Function Review: outboundTransfer(...)
1. Function Code
// Paste the exact outboundTransfer(...) source code here.
//
// Example structure:
//
// function outboundTransfer(
//     address _l1Token,
//     address _to,
//     uint256 _amount,
//     uint256 _maxGas,
//     uint256 _gasPriceBid,
//     bytes calldata _data
// ) external payable returns (bytes memory) {
//     ...
// }
Note:

The exact function body should be copied from the reviewed contract version.
Different gateway implementations may have different parameters and internal calls.
2. Function Purpose
outboundTransfer(...) starts the L1 -> L2 token deposit.

Its role is to connect:

user input;
token selection;
recipient selection;
amount accounting;
L1 escrow/lock logic;
calldata construction;
retryable ticket creation;
later L2 finalization.
In simple terms:

The user calls outboundTransfer(...) on L1 to begin bridging tokens to L2.
This function is security-critical because it turns a local L1 token action into
a cross-chain message that will later credit value on L2.

3. Critical Parameters
Common critical parameters:

l1Token / _l1Token
to / _to
amount / _amount
data / _data
gas limit
max gas / max submission cost
gas price bid / max fee per gas
refund addresses
Parameter meaning:

l1Token: source-chain token being deposited.
to: destination-chain recipient.
amount: requested deposit amount.
data: extra encoded parameters used by the bridge/gateway.
gas parameters: execution budget for the L2 retryable ticket.
refund addresses: addresses receiving unused retryable ticket funds.
4. Trusted / Untrusted Input
Trusted or semi-trusted:

configured gateway address;
configured inbox/bridge address;
validated token mapping;
expected counterpart gateway.
Untrusted:

user-supplied amount;
user-supplied to;
user-supplied data;
user-controlled gas values;
user-controlled refund addresses;
token contract behavior.
Important audit idea:

User input is not the same as economic reality.
The user can request to deposit amount, but the bridge must make sure the
source-chain accounting really supports the amount that will be credited on L2.

5. Internal Calls
Typical internal call structure:

outboundTransfer(...)
├── outboundEscrowTransfer(...)
├── getOutboundCalldata(...)
└── createRetryableTicket(...)
    └── AbsInbox._createRetryableTicket(...)
5.1 outboundEscrowTransfer(...)
Purpose:

Moves or locks the user's L1 tokens into bridge escrow.
Security meaning:

This is the accounting boundary.
The bridge must know how many tokens were actually received before it sends a
message that credits value on L2.

5.2 getOutboundCalldata(...)
Purpose:

Builds the calldata/payload that will be executed on L2.
Security meaning:

This is the message construction boundary.
The encoded amount, token, and recipient must match the real deposit.

5.3 createRetryableTicket(...)
Purpose:

Creates the L1 -> L2 retryable ticket.
Security meaning:

This is the cross-chain message boundary.
The retryable ticket must target the correct L2 gateway and carry correct
calldata.

6. Invariants
Main deposit invariant:

L1 locked / escrowed amount = L2 minted / released amount
Function-level invariants:

The selected L1 token must be the intended token.
The L1 token must map to the correct L2 token.
The recipient encoded for L2 must equal the intended recipient.
Tokens must be escrowed/locked before L2 credit is finalized.
The amount encoded for L2 must match the amount actually escrowed/received.
The retryable ticket must target the correct L2 gateway.
The deposit message must not be executable twice.
7. Amount Consistency
The amount should be tracked through the whole function and downstream flow:

input amount
-> actual received / escrowed amount
-> encoded amount
-> decoded amount on L2
-> minted / released amount
For non-standard tokens, the reviewed code should not blindly trust the
user-supplied amount.

Correct accounting formula:

actualReceived = balanceAfter - balanceBefore
Example risk:

User requested deposit: 100
Bridge actually received: 98
Amount encoded for L2: 100
L2 credited amount: 100
Broken invariant:

L1 escrowed amount < L2 minted / released amount
8. Security Risks
Risk 1: Amount mismatch
If the function uses the user-provided amount instead of the actual amount
received by escrow, L2 may credit more value than L1 received.

Impact:

Unbacked L2 tokens or bridge accounting insolvency.
Risk 2: Wrong token mapping
If the L1 token resolves to the wrong L2 token or wrong gateway, the user may be
credited with an unintended asset.

Impact:

Token identity corruption.
Risk 3: Recipient substitution
If the recipient is changed before calldata creation or during decoding on L2,
the deposit may be credited to the wrong address.

Impact:

Loss or redirection of user funds.
Risk 4: Unsafe user-controlled data
If _data can override critical values such as amount, token, recipient,
or refund addresses in an unsafe way, calldata may not represent the real
deposit.

Impact:

Calldata corruption and invalid L2 finalization.
Risk 5: Retryable ticket failure or griefing
If gas parameters are too low or refund logic is unsafe, the retryable ticket may
fail or become difficult to execute.

Impact:

Delayed or stuck deposit.
Risk 6: Wrong L2 target
If the retryable ticket targets the wrong L2 contract, the message may execute
unexpected logic or fail permanently.

Impact:

Invalid cross-chain execution.
9. Audit Questions
Who can call outboundTransfer(...)?
Which parameters are controlled by the user?
Is l1Token validated?
How is the correct L2 token/gateway selected?
Does escrow happen before calldata is created?
Does the function encode nominal amount or actualReceived?
Can fee-on-transfer tokens break accounting?
Can _data override amount, token, or recipient?
Can the recipient be substituted before finalization?
Does the retryable ticket target the expected L2 gateway?
Who controls refund addresses?
Can bad gas parameters cause a stuck deposit?
Is replay protection handled by the bridge/message system?
Can the same deposit be finalized twice?
10. Audit Conclusion
outboundTransfer(...) is the main L1 deposit entry point.

The function is critical because it connects user-controlled input with the
bridge accounting flow.

Main security boundary:

User input -> L1 escrow/accounting -> L2 message
Most important invariant:

The amount credited on L2 must be backed by the amount escrowed on L1.
Most important audit question:

Does the message sent to L2 represent what actually happened on L1?
If the function builds calldata from unverified user input, the destination chain
may credit value that is not properly backed on the source chain.

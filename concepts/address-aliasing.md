
# What is Address Aliasing?

It is a security transformation applied to the msg.sender when a Smart Contract sends a message from Layer 1 (Ethereum) to Layer 2 (like Arbitrum or Optimism).

## Graphic example
<img width="703" height="403" alt="image" src="https://github.com/user-attachments/assets/9d760fd2-01ae-497b-8587-127e7be533e9" />

# What actually happens without aliasing?????

## Description: A finalization ( finalizeInboundTransfer() ) receives the address which is equal the L1,but  the finalization uses the address which is already exits on L2, that is the wrong smart contract with the same address.

## Graphic example
<img width="1158" height="440" alt="image" src="https://github.com/user-attachments/assets/5df1ad4c-7876-43b7-9055-110c3e6e789d" />

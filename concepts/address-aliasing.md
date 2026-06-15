
# What is Address Aliasing?

It is a security transformation applied to the msg.sender when a Smart Contract sends a message from Layer 1 (Ethereum) to Layer 2 (like Arbitrum or Optimism).

## Graphic example
<img width="703" height="403" alt="image" src="https://github.com/user-attachments/assets/9d760fd2-01ae-497b-8587-127e7be533e9" />

# What actually happens without aliasing?????

## The function receives an L1 address, but on L2 this address may be another smart contract. This can lead to calling the wrong contract.

## Graphic example
<img width="1158" height="440" alt="image" src="https://github.com/user-attachments/assets/5df1ad4c-7876-43b7-9055-110c3e6e789d" />

# Arbitrum Bridge Flow Local Review
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/6f117924-6973-4ab4-baa8-bc4b01dcaad8" />


This repository contains my local security review and flow analysis of an Arbitrum-style token bridge.

The goal of this project is to understand how the bridge deposit and withdrawal flows work, how tokens are accounted for across L1 and L2, and where important security invariants can break.

This is an educational portfolio project. It is not an official audit of Arbitrum or any production deployment.

## What This Repository Covers

This repository focuses on:

- Deposit flow analysis
- Withdrawal flow analysis
- Function-by-function review
- Internal calls inside main bridge functions
- Token accounting
- Amount consistency
- Token mapping
- Recipient preservation
- Calldata and message passing
- Cross-chain authentication
- Replay protection
- Arbitrum address aliasing

## Review Methodology

The main methodology used in this review is:

```text
Understand the flow -> Define invariants -> Search for violations
For every important function, I analyze:

What the function does
Where it sits in the bridge flow
Which internal functions it calls
Which parameters are critical
Which inputs are user-controlled
What invariants must hold
What security risks may appear
What audit questions should be asked
Main Deposit Invariant
For deposits from L1 to L2:

L1 locked / escrowed amount = L2 minted / released amount
Main Withdrawal Invariant
For withdrawals from L2 to L1:

L2 burned / locked amount = L1 released amount
Repository Structure
The repository is organized by flow and by function.

Deposit flow files cover the L1 -> L2 path.

Withdrawal flow files cover the L2 -> L1 path.

Concept files explain important bridge-specific ideas such as Arbitrum address aliasing.

Purpose
The purpose of this repository is to show practical audit thinking for bridge systems.

Instead of only reading individual functions, the review follows the full token and message flow across chains and checks whether the bridge preserves:

amount
token identity
recipient
message authenticity
replay protection
accounting correctness
The main security question is:

Can destination-chain accounting diverge from source-chain reality?

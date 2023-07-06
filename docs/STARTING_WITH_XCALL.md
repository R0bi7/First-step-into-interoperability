# Starting with xCall

## Table of Contents

1. [Knowledge Required](#knowledge-required)
2. [Sending and Handling Cross-Chain Messaging](SENDING_CROSS_CHAIN_MESSAGE.md)
3. [Handling Fees](FEE_HANDLING.md)
4. [Handling Errors and Failures](ERROR_HANDLING.md)

## xCall

xCall is a standard interface to make permission-less calls between different blockchain networks.
Each network that ICON is interoperable with has an xCall contract address that dApps can call to transfer data across chains.
Currently, permission-less calls are done using the blockchain transmission protocol (BTP).

## Knowledge Required

### Smart Contracts

To successfully integrate xCall on the source and destination chains, basic smart contract skills are required.

Given the fact that xCall is currently supported on JVM (Icon, Havah) and EVM (Ethereum, BSC) chains, Java and Solidity smart contract development skills are necessary.

Here are some useful JVM and EVM smart contract programming links:
- [Java Smart Contracts on Icon](https://github.com/icon-community/icon.community/blob/6c4bd4ba602f99375d73f525d0e18249a48757f0/content/learn/java-articles/index.md)
- [Java Smart Contract Examples](https://github.com/icon-project/java-score-examples)
- [Solidity Language Documentation](https://docs.soliditylang.org/en/v0.8.20/)
- [Solidity by Example](https://solidity-by-example.org/)
- [BTP2 Wiki](https://github.com/icon-project/btp2/wiki)

### Off-chain Application

An off-chain application, often known as a dApp (decentralized application), is a piece of code that is responsible for coordinating interactions with the blockchain.

Client SDKs are frequently used to communicate with the blockchain efficiently.

For EVM:
- [Ethers](https://docs.ethers.org/v5/)
- [Web3.js](https://web3js.org/#/)

For JVM:
- [Icon Client APIs](https://docs.icon.community/icon-stack/client-apis)

You should be familiar with the aforementioned Client SDKs as well as the programming languages in which they are written.

---

## Do You Think You Have What It Takes?

Proceed to [Sending Cross-Chain Message](SENDING_CROSS_CHAIN_MESSAGE.md) to continue.

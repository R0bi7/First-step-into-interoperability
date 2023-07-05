# Starting with xCall

## Table of contents

1. [Knowledge required](#knowledge-required)
2. [Sending and handling cross chain messaging](SENDING_CROSS_CHAIN_MESSAGE.md)
3. [Handling fees](FEE_HANDLING.md)
4. [Handling errors and failures](ERROR_HANDLING.md)

## xCall

xCall is a standard interface to make permission-less calls between different blockchain networks.
Each network that ICON is interoperable with has an xCall contract address that dApps can call to transfer data across chains.
Currently, permission-less calls are done using the blockchain transmission protocol (BTP).

## Knowledge required

### Smart Contracts

To successfully integrate xCall on the source and destination chains, basic Smart Contract skills are required.

Given the fact that xCall is currently supported on JVM (Icon, Havah) and EVM (Etheruem, BSC) chains,
Java and Solidity Smart Contract development skills are necessary.

Useful JVM and EVM Smart Contract programming links:
- [Java Smart Contracts on Icon](https://github.com/icon-community/icon.community/blob/6c4bd4ba602f99375d73f525d0e18249a48757f0/content/learn/java-articles/index.md)
- [Java Smart Contract examples](https://github.com/icon-project/java-score-examples)
- [Solidity language documentation](https://docs.soliditylang.org/en/v0.8.20/)
- [Solidity by example](https://solidity-by-example.org/)
- [BTP2 Wiki](https://github.com/icon-project/btp2/wiki)

### Off-chain Application

An off-chain application, often known as a dApp (decentralised application), is a piece of code that is responsible for coordinating interactions with blockchain.

Client SDKs are frequently used to communicate with blockchain efficiently.

EVM:
- [Ethers](https://docs.ethers.org/v5/)
- [Web3.js](https://web3js.org/#/)

JVM:
- [Icon Client APIs](https://docs.icon.community/icon-stack/client-apis)

You should be familiar with before mentioned Client SDKs as well as programming languages in which they are written in.


---

## Do you think you got what it takes?

Proceed to [Sending cross-chain message](SENDING_CROSS_CHAIN_MESSAGE.md) to continue.
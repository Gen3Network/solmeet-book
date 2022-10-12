---
tags: notes
---

# A Complete Guide to Build a Simple Aggregator with Universal Rabbit Hole

***Updated at 2022.9.29***

> See the full example [here](https://github.com/DappioWonderland/universal-rabbit-hole-example)

## TL; DR

In this example, we will demonstrate **how to implement a simple aggregator** by using Universal Rabbit Hole, step by step.

#### Things you will be doing

- Instantiate DeFi instances in 1 line of code
- Switch to different DeFi protocol in 1 line of code
- Compose different DeFi protocols in less than 5 lines of code

#### Things you will NOT be doing

- You won't touch the details of various SDK
- You won't have to encode / decode the on-chain account data
- You won't have to figure out how to assemble the instruction

## Overview

**We want developers to use DeFi elements as Lego blocks.** Universal Rabbit Hole enables developers to utilize various DeFi elements, such as Pool, Farm, Vault, to compose more complex operations without handling enormous amount of client SDK and inconsistent interfaces. 

> See [this guide](https://guide.dappio.xyz/the-universal-rabbit-hole) and [this Medium Post](https://medium.com/dappio-wonderland/the-solution-to-composability-universal-rabbit-hole-28b817cc0fd4) for more details

### Architecture

There are two modules in Universal Rabbit Hole: **Navigator** and **Gateway**:

![](https://hackmd.io/_uploads/rJbWYMd-o.jpg)

#### Navigator (Reader)

Navigator is a Typescript client for instantiating various of kinds of DeFi protocols. You can use it as a standalone dependency in your own project or together with [Dappio Gateway](https://guide.dappio.xyz/the-universal-rabbit-hole).

#### Gateway (Writer)

Gateway is a **CaaS (Compasaility-as-a-Service)** that standardizes inter-protocol interaction on Solana to unlock the potential of composability. It has an universal interface for various Solana DeFi elements (Pool / Farm / MoneyMarket / Vault / Leveraged Farm / ...) and serves as a **common knowledge base** that helps Solana community learn and improve.

### Anatomy of Gateway

![](https://hackmd.io/_uploads/Skbcueoyi.jpg)

Let's take a closer look on Gateway module. There are 4 different components in order to make Gateway function:
  - **Builder**: Off-chain component that helps composing different DeFi actions
  - **Protocol(s)**: Off-chain component that packages the specific instruction set of each protocol
  - **Gateway**: On-chain program that receives and dispatches all the transactions, manages state, distributes fees
  - **Adapter(s)**: On-chain program that connects base program and Gateway program

While **Builder** and **Protocols** are packaged into a npm module, **Gateway (program)** and **Adapters** are Solana programs deployed by Dappio (sometimes third-party) to be invoked to interact with Base programs

### Workflow of Gateway

![](https://hackmd.io/_uploads/Bkvs9tU1i.png)

We can separate these series of operations into 2 distinct sections: **off-chain part** and **on-chain part**.

#### Off-chain Part

- User assembles actions by invoking action setter and providing proper parameters
- Builder composes transactions

#### On-chain Part

- User sends transactions directly to Gateway program
- Gateway dispatches transactions to their corresponding adapters through CPI (cross-program invocation)
- Adapter invokes Base program through another CPI

### Supported Protocols

Following are the current supported protocols:

| Program | ID | Type |
| - | - | - |
| Gateway | `GATEp6AEtXtwHABNWHKH9qeh3uJDZtZJ7YBNYzHsX3FS` | - |
| Adapter Raydium | `ADPT1q4xG8F9m64cQyjqGe11cCXQq6vL4beY5hJavhQ5` | Pool / Farm |
| Adapter Orca | `ADPTTyNqameXftbqsxwXhbs7v7XP8E82YMaUStPgjmU5` | Pool / Farm |
| Adapter Saber | `ADPT4GbWTs9DXxo91YGBjNntYwLpXxn4gEbxfnUPfQoB` | Pool / Farm |
| Adapter Lifinity | `ADPTF4WmNPebELw6UvnSVBdL7BAqs5ceg9tyrHsQfrJK` | Pool |
| Adapter Solend | `ADPTCXAFfJFVqcw73B4PWRZQjMNo7Q3Yj4g7p4zTiZnQ` | MoneyMarket |
| Adapter Francium | `ADPTax5HwQ2ZWVLmceCek8UrqMhwCy5q3SHwi8W71Kv2` | MoneyMarket |
| Adapter Larix | `ADPTLQQ1Bwgybb2qge7QKSW7woDrhEjcLWG642qP2X4` | MoneyMarket / Farm |
| Adapter Tulip | `ADPT9nhC1asRcEB13FKymLTatqWGCuZHDznGgnakWKxW` | MoneyMarket / Vault |
| Adapter Friktion | `ADPTzbsaBdXA3FqXoPHjaTjPfh9kadxxFKxonZihP1Ji` | Vault |
| Adapter Katana | `ADPTwDKJTizC3V8gZXDxt5uLjJv4pBnh1nTTf9dZJnS2` | Vault |
| Adapter NFTFinance | `ADPTyBr92sBCE1hdYBRvXbMpF4hKs17xyDjFPxopcsrh` | NFTFinance |

## Get Started

Follow the [offical documentation](https://hackmd.io/@DappioWonderland/complete-guide-to-buidl-solana-dapp-with-universal-rabbit-hole) of Universal Rabbit Hole to get started

## References

- https://guide.dappio.xyz/the-universal-rabbit-hole
- https://medium.com/dappio-wonderland/the-solution-to-composability-universal-rabbit-hole-28b817cc0fd4

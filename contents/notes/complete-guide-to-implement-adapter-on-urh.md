# SolMeet #14 - A Complete Guide to Implement an Adapter for Universal Rabbit Hole

**Author:** [@ironaddicteddog](https://twitter.com/ironaddicteddog), [@wei_sol_](https://twitter.com/wei_sol_), [@emersonliuuu](https://twitter.com/emersonliuuu)

***[Updated at 2022.11.24]***

- **Check our Public Beta [here](https://app.dappio.xyz)**
- Uiversal Rabbit Hole
    - [Navigator](https://github.com/DappioWonderland/navigator)
    - [Gateway](https://github.com/DappioWonderland/gateway)
    - [Adapter Programs](https://github.com/DappioWonderland/adapter-programs)
- The PR of the Genopets Integration
    - [Navigator PR](https://github.com/DappioWonderland/navigator/pull/112)
    - [Gateway PR](https://github.com/DappioWonderland/gateway/pull/15) 
    - [Adapter Programs PR](https://github.com/DappioWonderland/adapter-programs/pull/17)

## TL; DR

- What is Universal Rabbit Hole?
- Why is it necessary?
- How does it work?
- Demonstrate how to implement an adapter
  - Use Genopets farm as an example

## Overview

- Motivation
  - Open up the potential of composability
  - Develop higher level DeFi strategy without touching the implementation details
  - Attract more devs to build on top of your protocol
  - **NO NEED to re-invent the wheel (SDK)**
- Overview of Universal Rabbit Hole
  - Read part
    - Navigator
  - Writhe part
    - Gateway Program
    - Gateway Client
    - Adapter Program

#### Without URH

![](https://hackmd.io/_uploads/HykS0teHi.png)

- **Protocols are not composable**
- User has to deal with new SDK when trying to integrate new protocol

#### With URH

![](https://hackmd.io/_uploads/SJzgZC-Hj.png)

- **Protocols are composable**
- User only has to deal with builder (Gateway client) and Gateway program

### Workflow

![](https://hackmd.io/_uploads/rkZ6gRbSo.png)

### What is Genopets Staking

- Interfaces
  - Stake
  - Unstake
  - Harvest (With lockup period)

## Get Started

Follow the [offical documentation](https://docs.dappio.xyz/implemtation-guide-for-protocol-developers/example-2) of Universal Rabbit Hole to get started

## References

- https://app.dappio.xyz
- https://github.com/DappioWonderland/navigator
- https://github.com/DappioWonderland/gateway
- https://github.com/DappioWonderland/adapter-programs
- https://guide.dappio.xyz/the-universal-rabbit-hole
- https://github.com/genopets-solana

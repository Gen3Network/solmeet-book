---
tags: notes
---

# BUIDL an Orderbook-based DEX on Solana in 2 hours

**Author:** [@harry830622](https://twitter.com/harry830622), [@ironaddicteddog](https://twitter.com/ironaddicteddog)

***[Updated at 2022.08.25]***

> **See the example repo [here](https://github.com/harry830622/simple-serum)**

## TL; DR

DEX is an essential infrastructure for any DeFi ecosystem. Currently almost all exchanges in TradFi and CEXs are orderbook-based while major DEXs on Ethereum are AMMs due to some of Ethereum’s undesirable nature such as low TPS and high transaction fee, etc.

However, on Solana, a blockchain fundamentally more advanced than Ethereum, orderbook-based DEXs become much more practical.

In this session, we are going to BUIDL an orderbook-based DEX on Solana.

## What is Central Limit Order Book (CLOB)

From Wikipedia:

- A central limit order book (or **CLOB**) is a trading method used by most exchanges globally.
- It is a transparent system that matches customer orders (e.g. bids and offers) on a price time priority basis.
- The highest (best) bid order and the lowest (cheapest) offer order constitutes the best market or the touch in a given security or swap contract.
- Customers can routinely cross the bid/ask spread to effect immediate execution.

### CLOB vs AMM

| | CLOB | AMM |
| - | - | - |
| Depth Chart | ![](https://hackmd.io/_uploads/Bke21K34ko.png)<br>[Source](https://support.bullish.com/hc/en-us/articles/4554641116057-Reading-the-Depth-Chart#01G2QKAM4KHJBHNBW944B1NKWD) | ![](https://hackmd.io/_uploads/ByueYhVki.png)<br>[Source](https://medium.com/hackernoon/depth-chart-and-its-significance-in-trading-bdbfbbd23d33) |
| Price Discovery Mechanism | Match Engine | Constant Product Formula |
| Capital Efficiency | **High** | Low |
| Composability | Low | **High** |

## Serum: The Most Crucial Financial Infrastructure of Solana

![](https://hackmd.io/_uploads/ryjlB54Js.png)

- Serum DEX is the matching engine powering Solana-based financial projects
- Serum can power:
  - Spot Market
  - Derivatives Market (Zeta Markets / PsyOptions / off-piste / HXRO)
  - Liquidations (Jet Protocol / Tulip / Parrot)
  - Asset Management (Solrise / Nova Finance) 
  - AMMs (Raydium / Atrix / Cyclos)
  - In-game NFT markets (Star Atlas / DeFi Land / Aurory / OpenEra)

### Dominance of Serum

![](https://hackmd.io/_uploads/BkcYUtNJo.png)

- It takes **~30%** of instruction volume on Solana
- Some analytics:
  - [Program Instruction Volume](https://analytics.solscan.io/public/dashboard/8d888828-baae-47b9-948b-d087e5de1411?select_report_period=past7days)
  - [TVL](https://solscan.io/amm/serum)

### Ecosystem

![](https://hackmd.io/_uploads/BJ2KgT7Jj.png)

<!-- - Which protocols are building on top of Serum
- A scattering of ideas
  - AMM
  - borrow-lending protocol
  - margin trading
  - perpetual future
  - Options
  - ETF
  - Prediction Market
 -->

## Raydium: AMM Built on Top of CLOB

### Problem

- High gas fees
- Speed
- High slippage on large orders
- Fragmented liquidity

### Solution

From Raydium lightpaper:

> Unlike other AMM platforms, Raydium provides on-chain liquidity to a central limit order book, meaning that Raydium’s users and liquidity pools have access to the order flow and liquidity of the entire Serum ecosystem, and vice versa.

<!-- - The purpose of market makers is to facilitate the process of finding a fair price for a product as well as to provide market participants with a trading partner. 
- It is rewarded for each of these services. If the fair price is found, the market maker would earn the price difference between their bid and ask prices when participants trade against them.
- For providing liquidity the market maker earns a rebate fee from the exchange.  -->

- Raydium is a pure market maker which takes the tokens locked in it to create a series of orders at different price points and sizes to provide liquidity.
- It creates orders using the constant product invariant. This equation has the special property that it is stateless and given any two tokens, without any information about their relative prices or value, it can provide “infinite” liquidity to traders.
- Raydium utilizes this equation and prices orders on the orderbook according to the Fibonacci sequence to provide up to 20 orders at a variety of prices. 

## Components of Serum

- [Instruction Set (Program)](https://github.com/project-serum/serum-dex/blob/master/dex/src/instruction.rs#L327)
- [Instruction Set (Client)](https://github.com/project-serum/serum-ts/blob/master/packages/serum/src/instructions.js#L161)
- [Processor](https://github.com/project-serum/serum-dex/blob/master/dex/src/matching.rs)
- Main actions
  - PlaceOrder
  - MatchOrder
  - CancelOrder
  - SettleFund
  - ConsumeEvent

## Architecture

### Overview

![](https://hackmd.io/_uploads/SkFWNREks.png)

- Market is like a trading pair in CEX
    - Coin: Base Asset
    - PC(Primary Currency): Quote Asset
- Open Orders is like your account in CEX
- Vaults are the token accounts for the matching engine
- Bids/Asks compose an order book
- Request Queue is like a job queue
- Event Queue records the events such as orders filled, orders cancelled, etc.

### Place Order

![](https://hackmd.io/_uploads/S1wFrANyo.png)

- Side
    - Bid
    - Ask
- Order Type
    - Limit
    - Post Only
    - IOC
- Init an open orders if there is none
- Transfer funds to the vaults if the current deposit is not enough

### Match Order

![](https://hackmd.io/_uploads/Hk75rC4ks.png)

- Match the order with orders on the order book

### Cancel Order

![](https://hackmd.io/_uploads/Hy9or0EJj.png)

- Cancel the order placed before

### Settle Fund

![](https://hackmd.io/_uploads/Skz2r0NJo.png)

- Transfer funds out of the vaults to your own wallets

### Consume Event

![](https://hackmd.io/_uploads/Skt5rANyj.png)

- To know what happened in the DEX

## References

- [A technical introduction to the Serum DEX](https://docs.google.com/document/d/1isGJES4jzQutI0GtQGuqtrBUqeHxl_xJNXdtOv4SdII/view)
- https://uniswap.org/blog/uniswap-v3-dominance#comparison-of-uniswap-v3-versus-centralized-exchanges
- https://docs.projectserum.com/
- https://projectserum.medium.com/serum-srm-and-the-ecosystem-part-1-2742f6a24597
- https://projectserum.medium.com/serum-srm-and-an-ecosystem-for-the-future-part-2-91398e75ce27
https://projectserum.medium.com/calling-all-devs-serum-srm-and-an-ecosystem-for-the-future-part-3-ae7adbf4466e
- https://raydium.io/Raydium-Litepaper.pdf

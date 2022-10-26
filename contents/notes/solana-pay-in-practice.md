---
tags: notes
---

# Solana Pay in Practice: The Challenge and Solution

**Author:** [@akirawuc](https://twitter.com/akirawuc)

***[Updated at 2022.10.20]***

> **See the example repo:**
> - [solana-pay-escrow](https://github.com/akirawuc/solana-pay-escrow)
> - [solana-pay-merchant-side](https://github.com/akirawuc/solana-pay-escrow)

## TL; DR

- What is Solana Pay?
- What is the current challenge?
- How to improve it by a Escrow program?

<!-- 
## Goal

- How to use/integrate solana pay and use it in a better way
 -->

<!-- ## Agenda

- Overview of Solana Pay
    - Purpose
    - Mechanism
    - Method: transfer request
    - Live demo
    - Best practice
- What is the current challenge?
- Our Proposal to Improve Solana Pay
    - Transaction request
    - improvement of the transfer request
        - goal set
        - implementation
 -->

## Overview of Solana Pay

- What is Solana-pay? What can it do?
- What is the mechanism?
- How we can make it better?

### Quick Introduction

- What is Solana-pay?

> A standard protocol to encode Solana transaction requests within URLs to enable payments and other use cases. -- from their GitHub repo

### How Does It Work?

Basic version
We've mentioned that Solana pay is a protocol for payment, where the structure is as follow:
```
solana:<recipient>
       ?amount=<amount>
       &spl-token=<spl-token>
       &reference=<reference>
       &label=<label>
       &message=<message>
       &memo=<memo>
```

Web app to mobile wallet
![](https://hackmd.io/_uploads/Skmk5QLbi.png)

Web app to browser wallet
![](https://hackmd.io/_uploads/HJbb5XIZo.png)

Mobile app to mobile wallet
![](https://hackmd.io/_uploads/SyDbcQI-o.png)

### Executing Details

1. clone the repo, and run the example code in solana-pay/point-of-sales
```
#shell
git clone git@github.com:solana-labs/solana-pay.git
cd solana-pay/point-of-sales
```
2. directly demo how it works, and open up the console to show that how a transaction is confirmed.
```
npm install
npm run dev
npm run proxy
open "https://localhost:3001?recipient=Hw7QWUA98q8jUAgbHLGJsXycPESt6jiyF4cPFjgg5JVc&label=akirawu"
```

3. Create a transfer request


## Current Challenge

### Mechanism

Once a payment (a transfer request) on Solana pay is created, it will generate an address called `reference`, in order to check whether the payment is fulfilled.

System from the merchant side will continously check if there're transaction under the `reference` address, and will stop the checking if and only if the user fulfilled the payment. However, since the 'reference' is just a random address, it didn't have any preventing mechanism for customers to fulfilled it again.

### Problem: Repeat fulfilled, can use the same QR code to deposit as many times as you like

Essentially, the spirit of payment is a trade of value, where customer pay for merchant's product. Once the customer paid for the payment, the merchant will need to take the responsibility to either send product to the customer or refund. However, it faces difficulties in practice.

To analyze, we can separate a payment into 3 conditions, which are
1. complete (fulfilled exactly once)
2. repeated fulfilled (fulfilled multiple times)
3. incomplete (never fulfilled)

where we only need to discuss about the last 2:

- If the payment is repeated fulfilled:
    - Merchant can't prevent, and even have difficulty to notice it. By the mechanism currently, merchant's system will stop checking for the transaction under the 'reference' address once the payment is fulfilled.
    - The only possibility that the merchant can notice could only be either their system won't stop checking transaction under the 'reference' address or by checking every order manually themselves.
- If the payment is never fulfilled:
    - First, the check of the payment will continue forever, until being turned off manually. 
    - More importantly, if the order of the payment included a limited edition product, how can the merchant be able to handle that all manually?
        - if check quantity before payment create: how can they released those unpaid orders and prevent the customer to fulfilled it?
        - if check quantity after payment create: how to stop customers from paying after the product is sold out?

![](https://hackmd.io/_uploads/rJue7n8Nj.png)


## Our Proposal to Improve Solana Pay

<!-- Q: What can be improve?
A: Repeat deduction - the protocol won't be closed after the txn is done. -->

1. Time: will need to check if the transaction have been pending for a while
2. One time deposit: only the first time deposit should be success.

**The protocol have some time limit, and more specific is better, one payment only accept one time deposit.** We can use the new feature called **Transaction Request** on Solana Pay recently: 

```
solana:<url>
```

Instead of using transfer request to do the transfer, switch to use the transaction request.

### Architecture

#### Off-chain

![](https://hackmd.io/_uploads/rysB3dUVj.png)


![](https://hackmd.io/_uploads/rJYjghAXo.png)

#### On-chain

> Use the escrow program!

Check out: https://book.solmeet.dev/notes/intro-to-anchor
With the repo: https://github.com/ironaddicteddog/anchor-escrow

Notice that paying can just be consider as a single side escrow, while the initializer part won't need to transfer token into the vault, only the taker (buyer here) need to.

1. Initialize
![](https://hackmd.io/_uploads/S1srkgk4o.png)


2. Pay
![](https://hackmd.io/_uploads/B1-_1lyNs.png)


3. Cancel
![](https://hackmd.io/_uploads/rJ3tyeyVj.png)

<!-- ## Part 1: Implement `solana-pay-escrow` (WIP)

#### What is the problem with transfer?

#### Code: What's the different between escrow and solana-pay-escrow (the program here)

1. Initialize
```
```

2. Pay
```
```

3. Cancel
```
```

---

## Part 2: Implement `solana-pay-merchant-side` (WIP)

### Overview of the overall architecture

![](https://hackmd.io/_uploads/rkrcicpXj.png)


 -->

## References

- https://solmeet.dev/
- https://book.solmeet.dev/notes/intro-to-anchor
- https://github.com/ironaddicteddog/anchor-escrow
- https://github.com/solana-labs/solana-pay
- https://docs.solanapay.com/
- https://www.anchor-lang.com/
- https://coral-xyz.github.io/anchor/ts/index.html
- https://solana-labs.github.io/solana-program-library/token/js/

---
tags: notes
---

# Walk Through NFT Breeding Tokenomic and Program Design

<!-- > NFT 育種之代幣經濟與合約設計詳解

如何實作NFT育種與結合代幣經濟之典型實際案例應用

NFT breeding protocol workshop and case-study from popular application on solana. 
 -->

**Author:** [@web3lovemore](https://twitter.com/web3lovemore), [@emersonliuuu](https://twitter.com/emersonliuuu), [@wei_sol_](https://twitter.com/wei_sol_)

***[Updated at 2022.07.28]***

> **See the example repo [here](https://github.com/DappioWonderland/nft-breeding)**

## TL; DR

Deep diving on NFT application: How to do an breedable NFT and its application. For optimization of uility and playability.

(1) **Common Breeding Mechanisms** 

The application for breedable NFT with common mechanisms and its tokenomics. This session will be set as a basic breeding approach with verifiable randomness. Not only use as gamefi application but also extension and flexibility for new standard on metaplex.

Note that the **randomness** as blockhash method, the developer can be customized for ones ex. chainlink VRF 、 switchboard and Pyth nework etc.

(2) **Case Study**

Introducing popular cases and scenarios with breeding uility projects on Solana ex. StepN or Solchick. or even with Fungible token expense to breed the new NFT ex. Stoneapecrew.

This is can be standardized as breeding on-chain protocol for developer as the customized the uilities. 

## Overview

### NFT breeding mechanism and common design

note: A,B,C are NFT; $FT is FT.

(1) A+B->C    [Burn A,B]

(2) **A+B->A+B+C [Most common]**

(3) A+$FT->C  [tokenomics extension] 

(3) A+$FT->A+C 

(4) A+B+$FT->A+B+C 

### What is Breeding NFTs? 

Breeding NFTs is a mechanism in which two **breedable NFTs** breed to form a new NFT offspring. 

<!--Think of two animals breeding to create offspring to understand how the mechanism works.-->

NFT project creators and startups employ NFT breeding to create long-term utility for collectors of their NFT. Rather than depending on the ever-shifting value of NFTs based on public sentiment, project creators go a step further to create real methods and ways in which collectors can benefit from buying their NFTs. 

Breeding NFTs to get a new NFT (that's usually seen as more valuable and unique) is a significant way for collectors of an NFT to gain more from their initial investment. When a project uses NFT breeding to **create utility** for their NFTs, some or all of the NFTs in the collection of games are made to be breedable. 

Generally, each NFT in a collection or game is unique and distinguishable from others. This is how they're perceived as valuable. When the NFTs are breedable and breed with another breedable NFT, what results is a new NFT that did not previously exist. This NFT will possess a combination of the unique features (usually the best traits) of the parent NFTs.

Because of these unique features, the NFT offspring is a rarer and more unique NFT, **increasing the collector's portfolio value**.

### History of Breeding NFTs

With the advent of blockchain, gaming came breeding NFTs. CryptoKitties, besides being the first widely recognized blockchain game, first imbibed the concept of NFT breeding.

In the game, each Kitty is an NFT that's unique and valuable. Players can own and trade these Kitties just like holding other NFTs. Players can also breed these Kitties to generate offspring. 

Since the mechanism of NFT breeding became a successful and well-encouraged function in CryptoKitties, many other blockchain-based gaming projects have used NFT breeding to bring more utility to players who buy character NFTs in their games. 

![](https://hackmd.io/_uploads/S1Nr_Py69.png)

Photo Resource: https://www.youtube.com/watch?v=WbKVrhXfHaY

### Stories of Axie infinity: (Inspired by cryptoKitties)

see [Storis on Crypto kitty and Axie infinity feat. Animoca Brands](/HUWszmY-TeWb-MrqB637fg) for references

![](https://hackmd.io/_uploads/rk5JB6J65.png)
![](https://hackmd.io/_uploads/SkakS6yT5.jpg)

https://whitepaper.axieinfinity.com/gameplay/breeding

## Breeding New Tokens

With the advent of **NFT-based gaming** has come the breedable NFT. For anyone that likes online gaming, and creating new characters, breedable NFTs represent a tremendous opportunity. Every platform has its own rules for breeding, so it is hard to make blanket statements about how breedable NFTs work as a whole.

For example, **Roaring Leaders**, a new NFT collection, allows the Roaring Leaders NFTs to be bred, and produce Roaring Leaders cubs in a separate collection. The platform also created a novel, **Tinder-style dating system** that allows Roaring Leaders owners to **look for a 'mate' if they don't have both a male and female NFT to breed.**

Like other breedable NFTs, Roaring Leaders require payment in tokens to be bred. For this platform, the $ROAR token is used to pay for breeding.

![](https://hackmd.io/_uploads/HJU-dTypq.png)

## Breeding NFTs today

Apart from gaming projects, other NFT projects are now including NFT breeding as one of the utilities buyers of their NFTs will get.

This is irrespective of whether the NFTs are photos, videos, etc. Some NFT projects which utilize NFT breeding are **Rolling Leaders, Samurai Doge, and Fat Ape Club.**

Although the whole concept of breeding NFTs is universal among different NFT projects, the detailed workings of how it's implemented depend on the particular end goal of the project.

In some NFT projects, the parent NFTs will **be burned,** i.e., destroyed after breeding. In some other projects, the parent NFTs will not be burned. Instead, parent NFTs will be **prevented from breeding again **for some time after breeding.

In the same vein, for some projects, the resultant NFT is a combination of the parent NFTs only; In others, the consequent NFT possesses some unique features from each parent and some random features of its own, making it even rarer. **With NFTs, the rarer the features, the more unique the NFT is. And with NFT breeding, resultant NFTs are even rarer.**

For collectors of NFTs in a project, breeding NFTs serve to provide benefits beyond the aesthetic pleasure obtained from collecting NFTs.

#### DeRace[polygon]: NFT horse breeding with chainlink randomness:

![](https://hackmd.io/_uploads/BJj21Aypq.png)

Resource: https://deracing.org/learn/horses/derace-horse-breeding/

***More Tinder-Style example:**

ex. Crypto Punk and BAYC holder:RichBabies

![](https://hackmd.io/_uploads/Hyh8ITyp9.jpg)

Resource: https://opensea.io/collection/rich-baby

## Benefits of Breeding NFTs

For NFT collectors, being a part of projects that utilize breeding NFTs provides two main benefits.

One is the opportunity to own rarer and unique NFTs. In the NFT community, **the rarer the features of an NFT, the more valuable it's perceived to be**. Because of this, when a collector breeds two NFTs and gets a rarer offspring, he receives an NFT that's even **more valuable** than the parents he originally bought.

The second benefit is that collectors will be able to earn passive income through breeding NFTs. **Collectors can routinely breed NFTs to create rarer offspring and sell them while holding the parent NFTs they originally invested in.**

## Most Popular Breeding NFT Projects

1. CryptoKitties: (ETH)
2. Axie Infinity:  Two Axies breed to create new offspring with unique traits such as body parts, class, cards, etc. (Ronin; ETH side chain)
3. StepN (Solana)

## Case Study

Popular Solana project with nft breeding features: 

### (1) Solchicks

Only Gen0 NFTs, being the genesis collection, will have overall rarities (Common, Uncommon, Rare, etc.). 

There are certain restrictions to breeding. Each NFT will be able to breed a maximum of 7 times, referred to as Breeding Count which will be shown in the NFT’s metadata. **NFT breeding will also be subject to the Family Rule where NFTs with the same parents (either one of the parents) cannot breed with each other.** Furthermore, an NFT cannot breed with either of its parents. The Gen of the new NFT will always be one higher than that of the parent with the higher Gen.

<!--繁殖有一定的限制。每個 NFT 最多可以繁殖 7 次，稱為繁殖計數，將顯示在 NFT 的元數據中。NFT 繁殖也將受到家庭規則的約束，即具有相同父母（父母之一）的 NFT 不能相互繁殖。此外，NFT 不能與其父母中的任何一個一起繁殖。新 NFT 的 Gen 將始終比 Gen 較高的父代的 Gen 高一個。
-->

That means that if breeding occurs between a Gen 1 and another Gen 1, the resulting NFT will be Gen 2, but if the breeding occurs between a Gen 1 and a Gen 3, the resulting NFT will be Gen 4.

In terms of the cost for breeding, as mentioned above both $CHICKS and $SHARDS will be required to breed new SolChicks NFTs, and our initial principle of the breeding cost structure is as described below for the near-term, but we may modify this in the future. Rest assured that any modification to the core cost structure to the breeding system will be made in a transparent manner and will not be made lightly.

The cost of breeding will scale according to the parents’ generation (Gen) and Breeding Count. To explain further:
Breeding with Gen1 NFTs will be more expensive than breeding with Gen0 NFTs, breeding with Gen2 NFTs will be more expensive than breeding with Gen1 NFTs, and so on

Breeding with NFTs with 1 Breeding Count will be more expensive than breeding with 0 Breeding Count (i.e., the NFT has never been used for breeding), breeding with NFTs with 2 Breeding Count will be more expensive than breeding with 1 Breeding Count, and so on




![](https://hackmd.io/_uploads/rJ-Q8IT35.png)


Below is a table showing the required number of $CHICKS and $SHARDS for breeding for each parent NFT for different generations and remaining breed counts. 

<!--We will continuously be adjusting the required number of $CHICKS and $SHARDS tokens for breeding so as to loosely stabilize the floor price of Gen0 SolChicks NFTs.-->


![](https://hackmd.io/_uploads/H1DzjoJaq.png)


The utility of $CHICKS and $SHARDS will be as follows. Both tokens will be required for:

- Breeding new SolChicks NFTs
- Levelling up SolChicks NFTs
- Upgrading equipment and weapon NFTs
- Purchasing game items

Reference: 
[https://www.solchicks.io/](https://www.solchicks.io/) 
[https://whitepaper.solchicks.io/nft-breeding](https://whitepaper.solchicks.io/nft-breeding)

### (2) Stonedapecrew: 

*Retreats (V1)

We released the **first-ever on-chain NFT evolution process** in December 2021.

In version 1 of the NFT Evolution, the retreats, Chimpions can move up in the metaverse by going on retreat.

There they go on a deep reflection phase and with a bit of luck come back with a role. 

Two retreat options are available:

* Basic retreat for 333 $PUFF => 60% chance of getting a role
* Advanced DMT retreat for 666 $PUFF => features extra trips & chilled parties, therefore the chance of 80% for adopting a role.
* Ayahuasca Retreat for 1420 $PUFF => 100% chance of getting a role

![](https://hackmd.io/_uploads/SJrlV8T2q.png)

To go on a rescue mission:

Have 2 Apes with different roles
or rent one in our recruiting system for 5.25 SOL

Have enough $PUFF (start 1780 $PUFF)
every 100 rescued, it'll increase by 4.2%

reusing two apes increases the total cost by 60%
current cost (without reusing) can be viewed on https://www.stonedapecrew.com/rescue

![](https://hackmd.io/_uploads/BkuWsnkaq.png)

![](https://hackmd.io/_uploads/HJWwohJaq.png)

Reference: 
[https://www.stonedapecrew.com/](https://www.stonedapecrew.com/)
[https://docs.stonedapecrew.com/puff/utilities#awakening-v2](https://docs.stonedapecrew.com/puff/utilities#awakening-v2)

### (3) StepN (shoes-minting)

Shoe-Minting Event (SME) is when users use 2 Sneakers they own as a blueprint to “**breed”**, producing a Shoebox in the process. For reference, the 2 Sneakers will be called Vintages (Parents). Both Vintages need to be in the user’s possession (not under lease) and have full durability to begin an SME.

Users can then select a Sneaker, by heading to the **Mint** tab, choosing the Sneaker to “**breed**” with, and pressing **Mint** to proceed. The user will instantly receive a Shoebox that can be opened immediately.

#### *Dynamic Minting Costs
Minting cost = GST (A) + base GMT (B) + additional GMT ([A+B]*x)

1.    If GST < $4, x = 0%;
2.    If $4 < GST < $8, x = 50%;
3.    If $8 < GST < $12, x = 100%;
4.    If $12 < GST < $16, x = 200%;
5.    If $16 < GST < $20, x = 400%;
6.    If $20 < GST < $30, x = 800%;
7.    If $30 < GST < $40, x = 1600%;
8.    If $40 < GST < $50, x = 3200%;
9.    If GST > $50, x = 6400%.

[https://whitepaper.stepn.com/game-fi-elements/shoe-minting](https://whitepaper.stepn.com/game-fi-elements/shoe-minting)

Here are some **video demo ([1](https://drive.google.com/drive/folders/1YtFbJBODT_cu0JTA5myfCgCpOXQxV1Ll?usp=sharing), [2](https://www.youtube.com/watch?v=RixaU0eGlqk)) for nft breeding**

### Tokenomics (Extension)

![](https://hackmd.io/_uploads/r1qlh61T5.png)
resources: https://staratlas.com/

![](https://hackmd.io/_uploads/Hybwpakp5.png)

resources: https://www.genopets.me/ 

## Overview of NFT-Breeding Program

> #### Prerequisite: What is Metaplex Standard?
> >
> ![](https://i.imgur.com/yeO8vDC.png)
> image from [Metaplex docs](https://docs.metaplex.com/programs/token-metadata/accounts)

![](https://hackmd.io/_uploads/H1djPylTq.png)

### 1. Initialize

![](https://hackmd.io/_uploads/ry604xgpq.png)

- Store metaplex NFT attributes on chain
- Breeding Metadata
  - **Hash**
  - **Generation**
  - **Name**
  - **Metaplex Mint**
  - ParentA
  - ParentB
  - ...
  - AttributeA
  - AttributeB 
  - ...
- Initialization is **only** for genesis (Generation 0)
- Change Upgrade Authority of NFT to Breeding PDA signer so that the used NFT can be burnt by Breeding program

### 2. Compute New Data

![](https://hackmd.io/_uploads/S1k-rex6q.png)

- Customizable Breeding Logic with `compute` interface
- **Write attributes to `BreedingMeta` (PDA)**

### 3. Mint Child NFT

![](https://hackmd.io/_uploads/SJA7rlgpc.png)

- **Write Metaplex metadata (without URI)**
- Transfer upgrade authority to Breeding program PDA Signer
- Burn Parent NFTs(optional)
- Read attributes (off-chain)
- Upload image to web3 storage (off-chain)

### 4. Update URI of child NFT

![](https://hackmd.io/_uploads/B1ONSgl6q.png)

- Update URI in Metaplex metadata

<!-- ### Design Spec

See [NFT Breeding Spec](/5wGPnBffTCyYu1PZCs6KoA) for more details
 -->

## Reference

- https://solmeet.dev
- https://book.solmeet.dev
- https://t.me/joinchat/oDHbq6gwtZVkNjU1
- https://mirror.xyz/iamwgg.eth/sza6Vi5VyGtMweoQn-pja_hX9hEbqCdyOoJhJzM2qGI
- https://medium.com/cryptokitties/getting-started-with-cryptokitties-part-two-buying-and-breeding-792502e54a4d
- https://riseangle.com/nft-magazine/what-is-breeding-nfts-how-it-began-and-its-impact-on-nft-projects-today
- https://hackernoon.com/an-intro-to-breedable-nfts-and-social-connections-in-the-metaverse
- https://www.esports.net/crypto/breeding-games/
- https://hackernoon.com/breeding-nfts-like-pokemons-your-tokens-can-evolve
- [Tokenomics](/RU0jd4itRPeHm8i3MZtElg)
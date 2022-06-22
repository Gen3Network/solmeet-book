---
tags: notes
---

# Walk through Solana SDK design

**Author:** [@SaiyanBs](https://twitter.com/SaiyanBs), [@wei_sol_](https://twitter.com/wei_sol_), [@emersonliuuu](https://twitter.com/emersonliuuu)


***[Updated at 2022.06.22]***

## TL; DR

1. Why is SDK design important?
    * Usage - frontend
    * Performance
    * As connector between client side and programs
2. How to design SDK under Solana architecture
3. Compare between Bad & Good design

## Overview

<!-- ### (Front-end Implement @Benson )

### Frontend -->

> Get data and show it on the client side.

### How to Get the Data

#### web 2

1. Get data by API.
2. Mostly data is from the same or partnership company.

#### web 3
1. Get data by SDK.
2. Mostly need to get the on-chain data or others project open sourced SDK.

**Example**
```typescript=
// SDK
export async function getStakedAmount(
  poolInfos: PublicKey[],
  provider: anchor.Provider
) {
    ...
}
```
```yaml=
 // API
 /api/v1/getStakedAmount:
   get:
      tags:
      - "pet"
      summary: "Get staked amount"
      description: "Get staked amount"
      produces:
      - "application/xml"
      - "application/json"
      parameters:
      - name: "poolInfos"
        in: "query"
        description: ""
        required: true
        type: "array"
        items:
          type: "PublicKey"
        default: "[]"
      - name: "anchorProvider"
        in: "query"
        description: ""
        required: true
        type: "Provider"
        default: "[]"

```

### How to Show the Data on the Client Side

Basically it's the same no matter it's web2 or web3.
Nowadays most frontend developers choose frameworks to get the job done, and for now and especially on Solana, I believe Next.js is the best option.

#### Show it on the client side.

For instance, using Next.js, we need to split components like puzzles, split the whole page into pieces, and each piece may contain the content (layout and data).

And how we design the components **depends on**
1. layout (basically follows the design)
2. data flow
3. performance
4. maintenance (low coupling, readable)
5. how SDK or API design

Sometimes these points can against each other, so I feel like designing the components and data flow is more like an art, **may need to sacrifice some points to achieve another one**, so just choose a better way at the moment you did this.

#### For Instance, NFT staking page in [Dappio](https://app.dappio.xyz/nft-staking)
![](https://hackmd.io/_uploads/rks9rwOK5.png)
![](https://i.imgur.com/0iMhvPQ.jpg)

#### Discuss

1. What's the same part and different part between two pics ?
    * Different data by different project.
    * The pending reward card both exist in DappieGang and others project but with different layouts and position.
    * DappieGang has one utility row and filter part, but the others don't.
2. What's the similar funtion between two pics ?
    * Get staked info
    * Get pending reward
    * Claim pending reward
    * Stake / unstaked
3. What do we need to consider?

### Let's check the first part - Overview info
![](https://hackmd.io/_uploads/HkG_SEx5c.png)

```typescript=
// v1
export async function getStakedAmount(
  poolInfos: PublicKey[],
  provider: anchor.Provider
) {
    ...
}
```

As we can see, NFTs staked in pools which are accounts, so we need to know the address to get the account data.

How do we get the pool public key in v1 ? We hardcoded all of them.
```typescript=
const getAllStakedData = async () => {
      const allDappiePools = [
        ...nftStakingIDs.DAPG_COMMON_POOL_INFOS,
        ...nftStakingIDs.DAPG_LEGENDARY_PATTERN_POOL_INFOS,
        ...nftStakingIDs.DAPG_LEGENDARY_ROBOT_POOL_INFOS,
        ...nftStakingIDs.DAPG_LEGENDARY_ALIEN_POOL_INFOS,
        ...nftStakingIDs.DAPG_LEGENDARY_ZOMBIE_POOL_INFOS,
        ...nftStakingIDs.DAPG_GENESIS_POOL_INFOS,
      ];

      let res = 0;
      switch (props.checkingProject.projectName) {
        case ESupportedProjects.DAPPIEGANG:
          res = await getStakedAmount(allDappiePools, props.anchorProvider);
          break;
        case ESupportedProjects.SOVANA:
          res = await getStakedAmount(sovanaPools, props.anchorProvider);
          break;
        default:
          break;
      }
    
    ....
}
```

### Pending Reward
Then, it's the pending reward part which also exists in DappieGang's second row.
![](https://hackmd.io/_uploads/rk7EuNx99.png)

First thing we need to know is where's the reward from? The only reason we can get the `NFTU` is because we deposit our prove token into farm and farming/mining, so we need to know the farm's account to get the infos we need.

But where's the prove token from? The flow is 
1. We stake our NFTs into pools get prove token.
2. We deposit our prove tokens to get farming token and mining NFTU.

So we need to know which pool to stake first, because differnt NFTs rarity stake to different pools, and this is defined by initialization, and all the rarity info also stores on chain.

```typescript=
// Get user staked data
// Notice! This is all user staked, includes different projects
export async function getStakedNFTMint(
  owner: PublicKey,
  provider: anchor.Provider,
  poolInfo?: PublicKey
) {
    ...
}
    
// Then we need to know staked NFTs' rarity and pool info
// And in SDK v1, we have two options to get this

// Option 1, pass in the poolInfos we hardcoded at first 
export async function getPoolInfo(poolInfos: PublicKey[], provider: anchor.Provider) {
    ...
    // And this will return rarity and all the mints stored in the pool
    // Of course we need to map with user staked ones again to know the exact one.
}
    
// Option 2, pass in the mints we wanna know
export async function rarityFilter(mintList: PublicKey[], provider: anchor.Provider) {
    ...
    // And this will return what pool and rarity are about these mints.
    // And inside this SDK, it'll fetch the on chain program to get these data.
    // It could be a multiple RPC requests!!
}

    
// Next we need to get farm info by pool info
export async function getFarmFromPool(poolInfo: PublicKey, provider: anchor.Provider, nonce = 0) {
    ...
    // And this one only accept single poolInfo one time, so if user got multiple NFTs and staked in different pools, we need to call this funciton multiple times.
    // It could be a multiple RPC requests!!
}

    
// Finally we get the farm public keys, so we can get the pending rewards we want
export async function getUnclaimedReward(
  owner: PublicKey,
  farmInfo: PublicKey,
  provider: anchor.Provider
) {
    ...
    // Also, only accept one farm at a time, so could be
    // Multiple RPC requests!!
}
```

As we can see, just only the pending rewards could make tons of RPC requests.
And here we only go through the funtional parts, as a frontend developer, you need to deal with the component design to make it maintainable, readable, and make sure the performance won't destroy the user experience at the same time.
![](https://hackmd.io/_uploads/S1eXc-y59.png)

```typescript=
// And this is the SDK v1 stake, what should we do to make it happen ?
export async function stake(
  user: PublicKey,
  poolInfo: PublicKey,
  nftAccountList: PublicKey[],
  provider: anchor.Provider
) {
    ...
    // mint -> rarity -> pool -> stake
    // prove token -> farm -> deposit -> mining
}
```

#### Example - V2 version
```typescript=
export async function fetchAll(
  provider: anchor.Provider,
  adminKey?: PublicKey
) {
    ...
}
    
export function infoAndNftMatcher(
  allInfos: AllInfo[], 
  nftMint: PublicKey[]
) {
    ...
}
    
export function getStakedAmount(
  allInfos: AllInfo[],
  collection: string = "",
  rarity: string = ""
) {
  ...  
}
  
export async function stakeTxn(
  poolInfo: PoolInfo,
  user: PublicKey,
  userNftAccountList: PublicKey[],
  provider: anchor.AnchorProvider
) {
    ...
}
```

### Problems

1. **Hard code ID in SDK**

Which means once we add new category we will need to update SDK too. 
```
export const ALL_POOL_INFOS = [new PublicKey("POOL_INFO_KEY")];
```

And here's only part of the DappieGang infos, so as we partnership more projects, it'll end up become a file that you don't want to involved.

![](https://hackmd.io/_uploads/ry2BVi0Yq.png)

2. **Too many RPC request**
As we can see, in the pending rewards part, we need to call a tons of RPC requests to get the data we want, and it's just the pending reward part.
3. **Maintainance**
Again, by the pending reward example we know, you have to call functions one by one and sometimes not very intuitive, especially from the client side.

From a frontend developer's view, we need to deal with a lot of for loop, sync/async issues, components design, state management.

From a team member's view, with a poor SDK design, we need to setup a clear workflow for different roles (frontend, SDK, program).

For example, the hardcoded pool and farm infos, which side to store all of these infos, and when to update the file after initializing a new one, how to maintain this file ..etc

### Goal: Design the Most Friendly SDK or API for Frontend

- Stateless, less parameters or arguments.
- Call anywhere we want, no need to consider the context.

## SDK Design

### Solana system model

![](https://hackmd.io/_uploads/HJ7vkYHl5.png)

Reading from solana starts with sending a [http request](https://docs.solana.com/developing/clients/jsonrpc-api) to the RPC, and the data is sync from validator.

Writing data(sending transaction) to Solana is also first sent to RPC, the RPC will lookup the leader and pass the tx packet to it via a UDP request. The transaction will next be verified and processed.

Each instructions will be executed in the program, modifying account data. 
New block contained state changes will be sync across validators and voted.




### Storing data in Solana

[![](https://hackmd.io/_uploads/HkuU8iQx9.png)](https://paulx.dev/blog/2021/01/14/programming-on-solana-an-introduction)
> Image from https://paulx.dev/blog/2021/01/14/programming-on-solana-an-introduction/

### Good SDK Design: Rule of Thumbs

#### 1. SDK Architecture

![](https://hackmd.io/_uploads/H1I-g3aY5.jpg)

This is an overview of a good SDK design. It can be separated into two part, read and write.

The "read" part of the SDK is about fetching the data from RPC and deserialized into an object.

The "write" part of the SDK is to create a transaction object to interact with programs.

Utility, layouts and ids is used across reading and writing.

#### 2. Reduce RPC request

* Each Account data only needs to fetch once.
* Reduce the usage of `Connection`.

#### 3. Stateless design

* SDK is for reading/writing data to Solana.
* To read form the chain, account needs to be fetched and deserialized.
* To write to the chain, a TX is built and send.
* SDK only handles data decoding and tx building.
* Data is not modified by any functions/methods. 

## NFT Staking Program

![](https://hackmd.io/_uploads/S1OVTa0uq.png)

* A program that give out `Prove Token` by locking certain collections of NFT.
* NFT is stored separately in different Vaults.
* Mint list is manage by rarity program.

## NFT Staking Program (Implement)

> Find the full code base [here](https://github.com/Dappio-emerson/solmeet-9-nft-staking)

**Program Architecture**
```
solmeet-9-nft-staking
├── app
├── migrations
│   └── initialization
├── mintList
├── programs
│   ├── nft-rarity
│   └── nft-staking
├── target
│   ├── idl
│   └── types
├── tests
│   ├── v1
│   └── v2
└── ts
    ├── v1
    └── v2
```

### Setup Local Test Validator

Restart local validator, and clone a useful program for creating ATA from Mainnet. See more about the program [here](https://github.com/mercurial-finance/create-ata-if-missing-program)
```
$ solana-test-validator -r -c 9tiP8yZcekzfGzSBmp7n9LaDHRjxP2w7wJj8tpPJtfG -u https://api.mainnet-beta.solana.com
```

Configure RPC url to localnet and the wallet to the one for deploying. Make sure the wallet have enough balance (~ 5 SOL)
```
$ solana config get

# config solana setting to target wallet and network
$ solana config set -k ~/.config/solana/id.json -u <Localnet URL>

# check balance
$ solana balance

# request airdrop
$ solana airdrop 5
```


### Build And Deploy

Clone the code from github repo
```
$ git clone https://github.com/Dappio-emerson/solmeet-9-nft-staking.git
```

Install dependency and make sure you have replace program keys for all files below.
1. `programs/nft-rarity/src/lib.rs`
2. `programs/nft-staking/src/lib.rs`
3. `ts-v1/ids.ts`
4. `Anchor.toml`
```
#cd solmeet-9-nft-staking
$ yarn

# generate program key
$ anchor keys list
```

After replacing program keys, we are ready to build and deploy our program with command below.
```
$ anchor build
$ anchor deploy
```

**NOTICE:**
Make sure the **`Program Id`** you get after deployed match with the one in program **`declare_id!("6Utx...QnKM")`**, otherwise transaction we send might failed.

### SDK v1

Before we start staking NFT to program will need to initialize the allowed mint list to `rarity info` in rairty program, then initialize `pool info` with the `rarity info` key we just initialized. 

```
# run user defined scripts to initialize 
$ anchor run initializeState
```

#### Import library and declare variables

After initialization, we can implement staking part in `test/v1/1_nft-staking-v1.ts`, let's paste the code below to the file.

```typescript=
import * as anchor from "@project-serum/anchor";
import NodeWallet from "@project-serum/anchor/dist/cjs/nodewallet";
import { PublicKey } from "@solana/web3.js";
import * as fs from "fs";
import { findAssociatedTokenAddress } from "../../ts/v1/utils";
import * as nftFinanceSDK from "../../ts/v1";
import {
  COLLECTION_SEED,
  RARITY_SEED,
  MINT_LIST_PATH,
  connection,
} from "../0_setting";

describe("nft staking v1", () => {
  const wallet = NodeWallet.local();
  const options = anchor.AnchorProvider.defaultOptions();
  const provider = new anchor.AnchorProvider(connection, wallet, options);
  anchor.setProvider(provider);

  interface Classify {
    poolInfoKey: PublicKey;
    NftTokenAccountList: PublicKey[];
  }

  let poolInfoKey: PublicKey;
  let poolInfos: PublicKey[];
  let nftMintList: PublicKey[] = [];

  it("read nft mint", async () => {
    const rawData = fs.readFileSync(MINT_LIST_PATH, "utf-8");
    const data: string[] = JSON.parse(rawData);
    data.forEach((element) => {
      nftMintList.push(new PublicKey(element));
    });
  });

});

```

#### Add log before and after staking
```typescript=    
  it("read nft mint", async () => {
    ...
  }
  
  it("staked status: before stake", async () => {
    console.log("staked status: before stake");

    const poolInfos = await nftFinanceSDK.getPoolInfo([poolInfoKey], provider);

    console.log(
      `staking rate: ${(poolInfos[0].totalLocked / nftMintList.length) * 100}%`
    );
    console.log(`# of nft staked: ${poolInfos[0].totalLocked}`);
  });

  it("stake nft", async () => {
    // TODO
  });

  it("staked status: after stake", async () => {
    console.log("staked status: after stake");

    const poolInfos = await nftFinanceSDK.getPoolInfo([poolInfoKey], provider);

    console.log(
      `staking rate: ${(poolInfos[0].totalLocked / nftMintList.length) * 100}%`
    );
    console.log(`# of nft staked: ${poolInfos[0].totalLocked}`);
  });

  it("unstake nft", async () => {
    // TODO
  });
```

#### Implement stake/unstake 

Now, we are good to implement staking part!
Let's think about what we need for building stake transaction.

![](https://hackmd.io/_uploads/BkqjHagc5.png)

Arguments we need for staking:
- staking pool info
    - **pool info key**
    - prove token mint
    - prove token authority
    - ...
- user info
    - user address
    - NFT mint
    - prove token ATA

![](https://hackmd.io/_uploads/B1QbV3g5c.png)


Pool info key is one of the argument we're going to use later, so we need to generate it first.

```typescript=
  ...

  let poolInfoKey: PublicKey;
  let poolInfos: PublicKey[];
  let nftMintList: PublicKey[] = [];

  it("general pool info key", async () => {
    // find poolInfoAccount
    poolInfoKey = await nftFinanceSDK.getPoolInfoKeyFromSeed(
      wallet.publicKey,
      COLLECTION_SEED,
      RARITY_SEED,
      0
    );
  });

  it("read nft mint", async () => {
    ...
  }
```
Add stake and unstake test.
```typescript=
  ...
  
  it("stake nft", async () => {
    const pairs = await nftFinanceSDK.rarityFilter(nftMintList, provider);

    const pairsClassify: Classify[] = [];
    for (let pair of pairs) {
      const nftTokenAccount = await findAssociatedTokenAddress(
        wallet.publicKey,
        pair.mint
      );
      const target = pairsClassify.filter((item) =>
        item.poolInfoKey.equals(pair.poolInfoKey)
      );
      if (target.length == 0) {
        pairsClassify.push({
          poolInfoKey: pair.poolInfoKey,
          NftTokenAccountList: [nftTokenAccount],
        });
      } else {
        target[0].NftTokenAccountList.push(nftTokenAccount);
      }
    }

    for (let classify of pairsClassify) {
      console.log(`poolInfo: ${classify.poolInfoKey.toString()}`);
      const stakeTxn = await nftFinanceSDK.stake(
        wallet.publicKey,
        classify.poolInfoKey,
        classify.NftTokenAccountList,
        provider
      );
      for (let txn of stakeTxn) {
        const result = await provider.sendAndConfirm(txn, [wallet.payer]);
        console.log("<Stake>", result);
      }
    }
  });

  ...

  it("unstake nft", async () => {
    const userStakedNft = await nftFinanceSDK.getStakedNFTMint(
      wallet.publicKey,
      provider
    );

    const pairsClassify: Classify[] = [];
    for (let pair of userStakedNft) {
      const target = pairsClassify.filter((item) =>
        item.poolInfoKey.equals(pair.poolInfoKey)
      );
      if (target.length == 0) {
        pairsClassify.push({
          poolInfoKey: pair.poolInfoKey,
          NftTokenAccountList: [pair.nftMint],
        });
      } else {
        target[0].NftTokenAccountList.push(pair.nftMint);
      }
    }

    for (let classify of pairsClassify) {
      const unstakeTxn = await nftFinanceSDK.unstake(
        wallet.publicKey,
        classify.poolInfoKey,
        classify.NftTokenAccountList,
        provider
      );
      for (let txn of unstakeTxn) {
        const result = await provider.sendAndConfirm(txn, [wallet.payer]);
        console.log("<Unstake>", result);
      }
    }
  });
```
Now, we can run command below to see all test works well or not.
```
$ anchor run testV1
```

#### Implement stake/unstake transaction in SDK

After running test, you might found we didn't actually stake our NFT to program since we didn't implememt the logic for stake and unstake in SDK yet. Let's add the logic in `ts/v1/transaction.ts` and run the test again.

**Stake**
```typescript=
export async function stake(
  user: PublicKey,
  poolInfo: PublicKey,
  nftAccountList: PublicKey[],
  provider: anchor.Provider
) {
  anchor.setProvider(provider);
  const NftStakingProgram = new anchor.Program(
    nftStakingIDL,
    NFT_STAKING_PROGRAM_ID,
    provider
  );

  // fetch poolInfo
  const poolInfoAccount = await NftStakingProgram.account.poolInfo.fetch(
    poolInfo
  );

  // create user prove token ATA
  const userProveTokenAccount = await findAssociatedTokenAddress(
    user,
    poolInfoAccount.proveTokenMint
  );
  const createProveTokenAtaIx = await createATAWithoutCheckIx(
    user,
    poolInfoAccount.proveTokenMint
  );

  const createAtaIxArr: anchor.web3.TransactionInstruction[] = [];
  createAtaIxArr.push(createProveTokenAtaIx);

  const stakeTxArr: Transaction[] = [];
  for (let userNftAccount of nftAccountList) {
    const nftAccount = await getAccount(provider.connection, userNftAccount);
    const nftMint = nftAccount.mint;

    const [nftVaultAccount, _] = await PublicKey.findProgramAddress(
      [nftMint.toBuffer(), poolInfo.toBuffer(), Buffer.from(NFT_VAULT_SEED)],
      NftStakingProgram.programId
    );

    // create nft vault ATA
    let nftVaultAta = await findAssociatedTokenAddress(
      nftVaultAccount,
      nftMint
    );
    const createAtaIx = await createATAWithoutCheckIx(
      nftVaultAccount,
      nftMint,
      user
    );

    createAtaIxArr.push(createAtaIx);

    const StakeTx = NftStakingProgram.transaction.stake({
      accounts: {
        user,
        poolInfo,
        nftMint,
        userNftAccount,
        nftVaultAta,
        userProveTokenAccount,
        nftVaultAccount,
        proveTokenMint: poolInfoAccount.proveTokenMint,
        rarityInfo: poolInfoAccount.rarityInfo,
        proveTokenAuthority: poolInfoAccount.proveTokenAuthority,
        proveTokenVault: poolInfoAccount.proveTokenVault,
        systemProgram: anchor.web3.SystemProgram.programId,
        tokenProgram: TOKEN_PROGRAM_ID,
      },
    });
    stakeTxArr.push(StakeTx);
  }

  const createAtaTxArr: Transaction[] = [];
  let tx = new Transaction();
  for (let [index, ix] of createAtaIxArr.entries()) {
    tx.add(ix);
    if (
      (index + 1) % ATA_TX_PER_BATCH == 0 ||
      index == createAtaIxArr.length - 1
    ) {
      createAtaTxArr.push(tx);
      tx = new Transaction();
    }
  }
  const allTx = createAtaTxArr.concat(stakeTxArr);

  return allTx;
}
```

**Unstake**
```typescript=
export async function unstake(
  user: PublicKey,
  poolInfo: PublicKey,
  nftList: PublicKey[],
  provider: anchor.Provider
) {
  anchor.setProvider(provider);
  const NftStakingProgram = new anchor.Program(
    nftStakingIDL,
    NFT_STAKING_PROGRAM_ID,
    provider
  );

  // fetch poolInfo
  const poolInfoAccount = await NftStakingProgram.account.poolInfo.fetch(
    poolInfo
  );

  // create user prove token ATA
  const userProveTokenAccount = await findAssociatedTokenAddress(
    user,
    poolInfoAccount.proveTokenMint
  );

  const unstakeTxPromises = nftList.map(async (nftMint) => {
    const [nftVaultAccount, _] = await PublicKey.findProgramAddress(
      [nftMint.toBuffer(), poolInfo.toBuffer(), Buffer.from(NFT_VAULT_SEED)],
      NftStakingProgram.programId
    );

    let userNftAccount = await findAssociatedTokenAddress(user, nftMint);
    const createAtaIx = await createATAWithoutCheckIx(user, nftMint);

    // create nft vault ATA
    let nftVaultAta = await findAssociatedTokenAddress(
      nftVaultAccount,
      nftMint
    );

    const UnstakeTx = NftStakingProgram.transaction.unstake({
      accounts: {
        user,
        poolInfo,
        nftMint,
        userNftAccount,
        nftVaultAta,
        userProveTokenAccount,
        nftVaultAccount,
        proveTokenMint: poolInfoAccount.proveTokenMint,
        rarityInfo: poolInfoAccount.rarityInfo,
        proveTokenAuthority: poolInfoAccount.proveTokenAuthority,
        proveTokenVault: poolInfoAccount.proveTokenVault,
        tokenProgram: TOKEN_PROGRAM_ID,
      },
      preInstructions: [createAtaIx],
    });

    return UnstakeTx;
  });

  return Promise.all(unstakeTxPromises);
}
```

Run command again, then you can stake all your NFT to the program now!
```
$ anchor run testV1
```

### SDK v2
You might notice there are some problems while we implementing stake/unstake.
1. We need to know corresponding pool info key before we use it, and there are two way to get the key. One is generate with the seed as we done previous, and another is hard coded in SDK. Someone who used this SDK won't know the seed, so pool info key might need to hard coded in SDK and this result in frequently update SDK due to new partner joined.
2. Since we only have pool info key, we must fetch account info when building transaction. You can imagine if user is going to stake lots of NFT in different pool that might be a disaster for just building transaction by sending tons of RPC request.

#### Refactor SDK
In order to solve the issues above, we will try to
- Make stake/unstake transaction/instruction stateless
- Remove hard coded stuff

Let's take a closer look at v1 SDK, the root cause of why we need to make multiple RPC request comes from the bad design of the interface. If we **only** had `pool info key` associated with `PoolInfo`, we would **force** fetching data in every function that requires  `PoolInfo` data except the key. Actually, we can extract the fetching data from building transaction by implementing a fetch function, so called **`fetchAll()`**, to get all account we need for no matter matching NFT with pool or building transaction.

This is what `fetchAll()` function do, can see the `AllInfo` class definition in `ts/v2/poolInfos.ts`
```typescript=
export async function fetchAll(
  provider: anchor.Provider,
  adminKey?: PublicKey
): Promise<AllInfo[]> {
  const nftStakingProgram = new anchor.Program(
    nftStakingIDL,
    NFT_STAKING_PROGRAM_ID,
    provider
  );
  const nftRarityProgram = new anchor.Program(
    nftRarityIDL,
    NFT_RARITY_PROGRAM_ID,
    provider
  );

  let adminIdMemcmp: MemcmpFilter;
  if (adminKey != null && adminKey != undefined) {
    adminIdMemcmp = {
      memcmp: {
        offset: 8,
        bytes: adminKey.toString(),
      },
    };
  }

  const poolInfoSizeFilter: DataSizeFilter = {
    dataSize: nftStakingProgram.account.poolInfo.size,
  };
  let filters: (anchor.web3.MemcmpFilter | anchor.web3.DataSizeFilter)[] = [
    poolInfoSizeFilter,
  ];
  if (adminKey != null && adminKey != undefined) {
    filters = [poolInfoSizeFilter, adminIdMemcmp];
  }
  const allPoolInfos = await nftStakingProgram.account.poolInfo.all(filters);

  filters = [];
  if (adminKey != null && adminKey != undefined) {
    filters = [adminIdMemcmp];
  }
  const allRarityInfos = await nftRarityProgram.account.rarityInfo.all(filters);

  const allInfos: AllInfo[] = [];
  for (let currentRarityInfo of allRarityInfos) {
    for (let currentPoolInfo of allPoolInfos) {
      if (
        currentRarityInfo.publicKey.equals(currentPoolInfo.account.rarityInfo)
      ) {
        const rarityInfo = new RarityInfo(
          currentRarityInfo.publicKey,
          currentRarityInfo.account.admin,
          Buffer.from(currentRarityInfo.account.collection)
            .toString("utf-8")
            .split("\x00")[0],
          Buffer.from(currentRarityInfo.account.rarity)
            .toString("utf-8")
            .split("\x00")[0],
          currentRarityInfo.account.mintList
        );

        const poolInfo = new PoolInfo(
          currentPoolInfo.publicKey,
          currentPoolInfo.account.admin,
          currentPoolInfo.account.proveTokenMint,
          rarityInfo.key,
          currentPoolInfo.account.proveTokenAuthority,
          currentPoolInfo.account.proveTokenVault,
          Number(currentPoolInfo.account.totalLocked)
        );

        allInfos.push(new AllInfo(rarityInfo, poolInfo));
        break;
      }
    }
  }

  return allInfos;
}
```


After implementing `fetchAll()` with some class to store the data, now we only fetch data at the beginning by calling `fetchAll()`, then we can pass the data as an argument for building transaction.

![](https://hackmd.io/_uploads/BJrERYe5q.png)


#### Implement test with v2 (refactored) SDK

Open `tests/v2/1_nft-staking-v2.ts` you will see the code below (no need to do any modification). You can see that we don't need to generate pool info key first neither hard coded those keys in SDK now, we get all data with this line of code: **`allInfos = await nftFinanceSDK.fetchAll(provider);`**

```typescript=
import * as anchor from "@project-serum/anchor";
import NodeWallet from "@project-serum/anchor/dist/cjs/nodewallet";
import { PublicKey } from "@solana/web3.js";
import * as fs from "fs";
import { findAssociatedTokenAddress } from "../ts/utils";
import * as nftFinanceSDK from "../ts";
import { AllInfo } from "../ts/poolInfos";
import { UserInfo } from "../ts/userInfos";
import {
  COLLECTION_SEED,
  RARITY_SEED,
  MINT_LIST_PATH,
  connection,
} from "./0_setting";

describe("nft staking v2", () => {
  const wallet = NodeWallet.local();
  const options = anchor.AnchorProvider.defaultOptions();
  const provider = new anchor.AnchorProvider(connection, wallet, options);
  anchor.setProvider(provider);

  interface Classify {
    allInfo: AllInfo;
    NftTokenAccountList: PublicKey[];
  }

  let allInfos: AllInfo[];
  let nftMintList: PublicKey[] = [];

  it("read nft mint", async () => {
    const rawData = fs.readFileSync(MINT_LIST_PATH, "utf-8");
    const data: string[] = JSON.parse(rawData);
    data.forEach((element) => {
      nftMintList.push(new PublicKey(element));
    });
  });

  it("staked status: before stake", async () => {
    allInfos = await nftFinanceSDK.fetchAll(provider);
    console.log("staked status: before stake");

    const percentage = nftFinanceSDK.getStakedPercentage(
      allInfos,
      COLLECTION_SEED
    );
    console.log(`staking rate: ${percentage * 100}%`);
    const amount = nftFinanceSDK.getStakedAmount(allInfos, COLLECTION_SEED);
    console.log(`# of nft staked: ${amount}`);
  });

  it("stake nft", async () => {
    const pairs = nftFinanceSDK.infoAndNftMatcher(allInfos, nftMintList);

    const pairsClassify: Classify[] = [];
    for (let pair of pairs) {
      const nftTokenAccount = await findAssociatedTokenAddress(
        wallet.publicKey,
        pair.nftMint
      );
      const target = pairsClassify.filter((item) =>
        item.allInfo.poolInfo.key.equals(pair.allInfo.poolInfo.key)
      );
      if (target.length == 0) {
        pairsClassify.push({
          allInfo: pair.allInfo,
          NftTokenAccountList: [nftTokenAccount],
        });
      } else {
        target[0].NftTokenAccountList.push(nftTokenAccount);
      }
    }

    for (let classify of pairsClassify) {
      const stakeTxn = await nftFinanceSDK.txn.stakeTxn(
        classify.allInfo.poolInfo,
        wallet.publicKey,
        classify.NftTokenAccountList,
        provider
      );
      for (let txn of stakeTxn) {
        const result = await provider.sendAndConfirm(txn, [wallet.payer]);
        console.log("<Stake>", result);
      }
    }
  });

  it("staked status: after stake", async () => {
    console.log("staked status: after stake");
    allInfos = await nftFinanceSDK.fetchAll(provider);

    const percentage = nftFinanceSDK.getStakedPercentage(
      allInfos,
      COLLECTION_SEED
    );
    console.log(`staking rate: ${percentage * 100}%`);
    const amount = nftFinanceSDK.getStakedAmount(allInfos, COLLECTION_SEED);
    console.log(`# of nft staked: ${amount}`);
  });

  it("get user info", async () => {
    const userInfo = await nftFinanceSDK.fetchUser(wallet.publicKey, provider);

    console.log(`user address: ${userInfo.wallet.toString()}`);
    console.log(`# of user staked nft: ${userInfo.staked.length}`);
  });

  it("unstake nft", async () => {
    const userInfo = await nftFinanceSDK.fetchUser(wallet.publicKey, provider);
    const pairsClassify: Classify[] = [];
    for (let pair of userInfo.staked) {
      const target = pairsClassify.filter((item) =>
        item.allInfo.poolInfo.key.equals(pair.poolInfoKey)
      );
      if (target.length == 0) {
        pairsClassify.push({
          allInfo: nftFinanceSDK.getAllInfoFromPoolInfoKey(
            allInfos,
            pair.poolInfoKey
          ),
          NftTokenAccountList: [pair.nftMint],
        });
      } else {
        target[0].NftTokenAccountList.push(pair.nftMint);
      }
    }

    for (let classify of pairsClassify) {
      const unstakeTxn = await nftFinanceSDK.txn.unstakeTxn(
        classify.allInfo.poolInfo,
        wallet.publicKey,
        classify.NftTokenAccountList,
        provider
      );
      for (let txn of unstakeTxn) {
        const result = await provider.sendAndConfirm(txn, [wallet.payer]);
        console.log("<Unstake>", result);
      }
    }
  });
});
```

Run command below to make sure no issue occured.
```
$ anchor run testV2
```


#### Implement stake/unstake transaction in SDK v2
Once all test passed, let's add the logic for stake and unstake transaction in `ts/v2/transaction.ts`. Since we pass in full pool info data instead of only the key, we can remove all RPC request during transaction building.

**Stake**
```typescript=
export async function stakeTxn(
  poolInfo: PoolInfo,
  user: PublicKey,
  userNftAccountList: PublicKey[],
  provider: anchor.AnchorProvider
) {
  const ixArr: anchor.web3.TransactionInstruction[] = [];
  const createAtaTxnArr: Transaction[] = [];
  const stakeTxnArr: Transaction[] = [];
  for (let [index, userNftAccount] of userNftAccountList.entries()) {
    const StakeIxArr = await ix.stakeIx(
      poolInfo,
      user,
      userNftAccount,
      provider
    );
    if (index == 0) {
      ixArr.push(StakeIxArr[StakeIxStatus.createUserProveTokenAtaIx]);
    }
    ixArr.push(StakeIxArr[StakeIxStatus.createNFTVaultAtaIx]);
    const stakeTx = new Transaction();
    stakeTx.add(StakeIxArr[StakeIxStatus.stakeIx]);
    stakeTxnArr.push(stakeTx);
  }
  let txn = new Transaction();
  for (let [index, instruction] of ixArr.entries()) {
    txn.add(instruction);
    if ((index + 1) % ATA_TX_PER_BATCH == 0 || index == ixArr.length - 1) {
      createAtaTxnArr.push(txn);
      txn = new Transaction();
    }
  }

  const allTxn = createAtaTxnArr.concat(stakeTxnArr);

  return allTxn;
}
```

**Unstake**
```typescript=
export async function unstakeTxn(
  poolInfo: PoolInfo,
  user: PublicKey,
  nftMintList: PublicKey[],
  provider: anchor.AnchorProvider
) {
  const allTxn: Transaction[] = [];

  for (let nftMint of nftMintList) {
    const createAtaIx = await createATAWithoutCheckIx(user, nftMint);
    const unstakeIx = await ix.unstakeIx(poolInfo, user, nftMint, provider);
    const txn = new Transaction();
    txn.add(createAtaIx);
    txn.add(unstakeIx);
    allTxn.push(txn);
  }

  return allTxn;
}
```

Run command below again and now we successfully use refactored SDK to stake NFT!
```
$ anchor run testV2
```

### Difference Between v1 and v2 SDK
- **How we get account data**
In v1 we fetch one account since we only get one pool info key at a time, but in v2 we done the fetching part at the beginning, and fetch all account with same structure in one RPC request.
- **Where we get account data**
In v1 we hard coded the account address in SDK instead of storing data in client side by fetching all account we need at a time. In v2 we implement with the opposite way, which reduce the frequency of fetching same account in different function.
- **Maintainability**
In v1 we'll need to update the hard coded stuff if new partner joined or we add new category which is tough to maintain. In v2, by replacing hard coded stuff with storing class in client side, there's no need to modify SDK due to new pool been created.

## Reference
- [book.solmeet.dev](https://book.solmeet.dev/)
- [BUIDL an Auto-compounding Bot on Saber](https://book.solmeet.dev/notes/buidl-auto-compounding-bot)
- [Dappio: NFT staking](https://app.dappio.xyz/nft-staking)
- [Anchor Book](https://book.anchor-lang.com/)
- [JSON RPC API](https://docs.solana.com/developing/clients/jsonrpc-api)
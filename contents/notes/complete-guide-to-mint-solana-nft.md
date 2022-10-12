---
tags: notes
---

# A Complete Guide to Mint Solana NFTs with Metaplex

**Author:** [@ironaddicteddog](https://twitter.com/ironaddicteddog)

***[Updated at 2022.4.27]***

> **See the example repo [here](https://github.com/DappioWonderland/nft-mint-example)**

## Overview

- Generate profile pictures (pfp) from trait materials with configurable weights
- Use Metaplex Standard
- Upload pfp and metadata to Arweave, which is a decentralized storage network
- Mint NFT on [`solana-mf`](https://github.com/DappioWonderland/solana), a mainnet-fork developed by Dappio
- Some handy tools
  - `hashlips_art_generator`
  - `arweave-image-uploader`
  - `metaboss`
 
## Introduction to Metaplex

- What does Metaplex do?
  - Metaplex Standard is adopted by popular wallets such as Phantom
- What is in Metaplex standard?
  - See [here](https://medium.com/metaplex/metaplex-metadata-standard-45af3d04b541) for more details

<!-- - Metaplex Standard vs Metaplex Tool
- What metadata should be provided?
- Metaplex tool is not very handy to use
- TODO: cons -->

## Setup

### Structure

```
├── 📂 solmeet-3-sandbox
│
├── 📂 hashlips_art_engine
│   │
│   ├── 📂 layers
│   │
│   └── 📂 build
│       │
│       ├── 📂 images
|       |
|       └── 📄 _metadata.csv
│
├── 📂 arweave-image-uploader
│   │
│   └── 📂 public
│       |
│       ├── 📂 images
│       |
│       ├── 📄 data.csv
│       |
│       └── 📄 arweave-uris.json
│
└── 📂 mint
    │
    ├── 📄 mint.js
    │
    └── 📄 mints.json
```

### This Tutorial Only Works on x86_64 Chip

> See [here](https://github.com/Automattic/node-canvas/issues/1733) for more discussion and work arounds

This tutorial only works on x86_64 chip and **does not work on Apple Sillicon (M1 Chip)**. Some C++ libraries (ex: `cairo`) may fail. If you are on M1 chip, I strongly recommend you to use a Linux VPS. Here are some options:
- [DigitalOcean](https://www.digitalocean.com)
- [Linode](https://www.linode.com)
- [Vultr](https://www.vultr.com)

### Install `rust`

- See [this doc](https://hackmd.io/@ironaddicteddog/solana-starter-kit#Install-Rust-and-Solana-Cli) for more details

### Install `solana`

- See [this doc](https://hackmd.io/@ironaddicteddog/solana-starter-kit#Install-Rust-and-Solana-Cli) for more details

### Download `solmeet-3-sandbox`

#### Option 1: Use Google Drive Web

Folder link [here](https://drive.google.com/drive/folders/1tIXmkRq0cKLp6BZ_hP5_JjcdHa-WKpZV?usp=sharing)

#### Option 2: Use `gdown` (For Ubuntu User)

Install `gdown`:

```bash
$ sudo apt update
$ sudo apt install python3-pip
...

$ pip install gdown
...

```

Download `background`, `base`, `clothes`, `faces`, `hats` separately:

```bash
$ mkdir solmeet-3-sandbox
$ cd solmeet-3-sandbox
$ gdown --folder https://drive.google.com/drive/folders/1RLz4J7TTh9cnXKWJlUb6_SC5dSnDYiBL -O background
...

$ gdown --folder https://drive.google.com/drive/folders/1jj4V7GNvFqc2UROZhaEvoaF1t8vP53TF -O base
...

$ gdown --folder https://drive.google.com/drive/folders/1FXuztlvSfIsStFXu4_dInXwV9xz_b-gz -O clothes
...

$ gdown --folder https://drive.google.com/drive/folders/1TM5zK9pHm73oSO1U8hpg6G1An14cyagU -O faces
...

$ gdown --folder https://drive.google.com/drive/folders/1GKYw77k0gQRX-AbtTtNChzpGsCBNL1bJ -O hats
...

```

#### Option 3: Use `scp`

Download the folder to local machine first

```bash
$ scp -r [path of the local folder] user@host:[path to s]
```

### Install `hashlips_art_engine`

- https://github.com/HashLips/hashlips_art_engine

```bash
$ git clone https://github.com/HashLips/hashlips_art_engine.git
...

$ cd hashlips_art_engine
$ yarn
...

```

> Notice: Make sure your `node` version >= v16.13.0.  See this [issue](https://github.com/HashLips/hashlips_art_engine/issues/375) for more details.

### Install `arweave-image-uploader`

- https://github.com/thuglabs/arweave-image-uploader

```bash
$ git clone https://github.com/thuglabs/arweave-image-uploader.git
...

$ cd arweave-image-uploader
$ yarn
...

```

### Install `metaboss`

- https://github.com/samuelvanderwaal/metaboss

```bash
$ sudo apt-get install pkg-config libssl-dev libudev-dev
...

$ cargo install metaboss --locked
...

```

### Install `proxyman`

- https://proxyman.io/release/osx/Proxyman_latest.dmg

**This should be installed on your local machine.**

> For Linux / Windows developers, you could choose whistle (open source) or postman.

### Setup Arweave Wallet

Follow this [doc](https://docs.arweave.org/info/wallets/arweave-web-extension-wallet) to setup your Arweave wallet and claim free AR token by completing assigned [task](https://faucet.arweave.net/).

**You should have a downloaded key file after the setup.** We will need the keyfile in the rest of the tutorial.

## Part 1: Generate Art Works

### Modify `hashlips_art_engine`

Additionally, we have to make a few small changes in the codebase to export the data with desired format.

Next, **replace the source code of `hashlips_art_engine/src/main.js` with the code from the [example](https://raw.githubusercontent.com/DappioWonderland/nft-mint-example/master/hashlips_art_engine/src/main.js)**.

Here are the changes we made:

```javascript
// In src/main.js

...

// Line 33
let traits = layerConfigurations[0].layersOrder.map(o => o.name);
let metadataListCsv = [`Name,${traits.join(",")}`];
...

// Line 171
metadataListCsv.push(`${tempMetadata.name.split('#')[1]},${attributesList.map(o => o.value).join(",")}`);
...

// Line 317
const writeMetaDataCsv = (_data) => {
  fs.writeFileSync(`${buildDir}/_metadata.csv`, _data);
};
...

// Line 441
writeMetaDataCsv(metadataListCsv.join('\n'));
...

```

### Config `hashlips_art_engine`

**Replace the source code of `hashlips_art_engine/src/config.js` with the code from the [example](https://raw.githubusercontent.com/DappioWonderland/nft-mint-example/master/hashlips_art_engine/src/config.js)**.

Here are the changes we made:

```javascript
// In src/config.js

...

// Line 5
const network = NETWORK.sol;
...

// Line 8
const namePrefix = "";
...

// Line 25
const layerConfigurations = [
  {
    growEditionSizeTo: 10,
    layersOrder: [
      { name: "background" },
      { name: "base" },
      { name: "clothes" },
      { name: "faces" },
      { name: "hats" },
    ],
  },
];
...

```

### Build Images

Copy `solmeet-3-sandbox` to `hashlips_art_engine`:

```bash
$ cd hashlips_art_engine
$ rm -rf ./layers
$ cp -r ../solmeet-3-sandbox ./layers
```

Set the rarity for each trait **by adding a weight number in filename**. In this tutorial, we will keep every value the same weight in a certain trait.

After updating the filenames, you should have the following results:

```bash
$ ls layers/background
bg1#1.png  bg2#1.png  bg3#1.png  bg4#1.png  bg5#1.png

$ ls layers/base
base1#1.png  base2#1.png

$ ls layers/clothes
clothes1#1.png  clothes2#1.png  clothes3#1.png  clothes4#1.png  clothes5#1.png

$ ls layers/faces
face1#1.png  face2#1.png  face3#1.png  face4#1.png  face5#1.png

$ ls layers/hats
hat1#1.png  hat2#1.png  hat3#1.png  hat4#1.png  hat5#1.png
```

Finally, build the images:

```bash
$ yarn run build
```

This will export the images to `images` folder and a `_metadata.csv` file, both under `build` folder.

> Note: You can compute the distribution of rarity by this command:
> ```bash
> $ yarn run rarity
> ```

## Part 2: Upload to Arweave

### Setup Arweave Wallet

Follow this [doc](https://docs.arweave.org/info/wallets/arweave-web-extension-wallet) to setup your Arweave wallet and claim free AR token by completing assigned task.

**You should have a downloaded key file after the setup.** We will need the keyfile in the rest of the tutorial.

### Modify `arweave-image-uploader`

Install `dotenv`:

```bash
$ yarn add dotenv
```

Copy the whole string from the downloaded key file and paste to new `.env` file:

```bash
$ touch .env
```

```
// In .env

KEY={"kty":"RSA","n":"tS1op66z_hQcHj5rKo_WZPvQp3nUP-auQCHqMr..."}
```

**Replace the source code of `arweave-image-uploader/uploader.js` with the code from the [example](https://raw.githubusercontent.com/DappioWonderland/nft-mint-example/master/arweave-image-uploader/uploader.js)**.

Here are the changes we made:

```javascript
// In uploader.js

...

// Line 6
import dotenv from "dotenv";
dotenv.config();
...

// Line 21
const getNftName = (name) => `SolMeet-3 ART #${name}`;

const getMetadata = (name, imageUrl, attributes) => ({
  name: getNftName(name),
  symbol: "SMT",
  description:
    "SolMeet #3 Art Work",
  seller_fee_basis_points: 100,
  external_url: "https://solmeet.dev",
  attributes,
  collection: {
    name: "SolMeet",
    family: "Dev",
  },
  properties: {
    files: [
      {
        uri: imageUrl,
        type: "image/png",
      },
    ],
    category: "image",
    maxSupply: 0,
    creators: [
      {
        address: "DaPYbGagq3dFDZ1i2PWpSP27mg1ty7J3XfQQciQPLsUn",
        share: 100,
      },
    ],
  },
  image: imageUrl,
});
...

// Line 93
let key = JSON.parse(process.env.KEY);
...

// Line 123
let metadataUri = [];
let metadataCollectionUri = [];
...

// Line 181
metadataUri.push(metadataUrl);
...

// Line 192

// Collection
const collectionFilePath = folder + "logo.png";
const collectionLogo = fs.readFileSync(collectionFilePath);
const contentType = ["Content-Type", "image/png"];
const { id } = await runUpload(collectionLogo, contentType, true);
const imageUrl = id ? `https://arweave.net/${id}` : undefined;
const collectionName = "SolMeet NFT DAO";
const collectionFamily = "DAO";
const metadata = getCollectionMetadata(
  collectionName,
  collectionFamily,
  imageUrl
);
const metaContentType = ["Content-Type", "application/json"];
const metadataString = JSON.stringify(metadata);
const { id: metadataId } = await runUpload(metadataString, metaContentType);
const metadataUrl = id ? `https://arweave.net/${metadataId}` : undefined;

console.log("metadataUrl", metadataUrl);
const newItem = {
  collection: {
    name: collectionName,
    uri: metadataUrl,
  },
};
metadataCollectionUri.push(metadataUrl);

metadataCollection = { ...metadataCollection, ...newItem };
...

// Line 227
const uris = JSON.stringify(metadataUri);
fs.writeFileSync("./public/arweave-uris.json", uris);
const collectionUris = JSON.stringify(metadataCollectionUri);
fs.writeFileSync("./public/arweave-collection-uris.json", collectionUris);
...

```

**Notice: make sure that the address of `creators`** is the same as the mint transaction sender, which is the Solana cli wallet. You can double check via this command:

```bash
$ solana address
DaPYbGagq3dFDZ1i2PWpSP27mg1ty7J3XfQQciQPLsUn
```

### Upload Images

Copy the images and metadata to `arweave-image-uploader`:

```bash
$ cd arweave-image-uploader
$ rm -rf public/images
$ cp -r ../hashlips_art_engine/build/images public/images
$ cp ../hashlips_art_engine/build/_metadata.csv public/data.csv
```

Upload to Arweave:

```bash
$ yarn run upload
...

```

After uploading, you should see a output file `arweave-uris.json`  under `public` folder. This is the uris of all the metadata. We will soon use it for minting in the next step.

Also, you can access your image and metadata by visiting the uri. For example:
- https://arweave.net/hcekmHUHRlQhTHSh0wc0m7zL_EyUahr_IjGlBk-4EO8
- https://viewblock.io/arweave/tx/hcekmHUHRlQhTHSh0wc0m7zL_EyUahr_IjGlBk-4EO8

## Part 3: Mint NFTs

### Config Solana

Here, we use `solana-mf` for deploying and testing:

```bash
$ solana config set --url https://rpc-mainnet-fork.dappio.xyz
...

$ solana config set --ws wss://rpc-mainnet-fork.dappio.xyz/ws
...
```

Don't forget to request for airdrop at the first place:

```bash
$ solana airdrop 1
...

```

### Mint

We will use `metaboss` for interacting with Metaplex. Here we have to do 3 things in order:
- Create Collection
- Mint NFTs
- Set and verify the Collection for NFTs

First, create a folder `mint` an empty file `mint.js`:

```bash
$ mkdir mint
$ cd mint
$ touch mint.js
```

Next, **replace the source code of `mint/mint.js` with the code from the [example](https://raw.githubusercontent.com/DappioWonderland/nft-mint-example/master/mint/mint.js)**.

Then, run the minting script:

```bash
$ KEYPAIR=~/.config/solana/id.json RECEIVER=MY_ADDRESS AUTHORITY=MY_ADDRESS node mint.js 

collectionMint: 6NyVbJX1TB9HMYs5Wfx7qDuPcj8gX55DgsPh97Fv3M8T
nftMint: 6ofefmhENgvXwPznjJikZiAsbEKbxZb689gtkRcR4gUu
nftMint: HicQBruRwJHr7peuzAvZyhUrmwAAYXppjX1VQNVF15CC
nftMint: 2h7rHru4WRzBJnw3M7dN8TSfoUZPnzqD1Beo53gm6pC9
nftMint: 8vD2rNDNs6VZ7H4cRuRmi7uwZEff2MXxMA1Cbq9KdTm1
nftMint: 747BYudqdx9zdvwhvwvn5j4MDekpsuv9dEL5zNewcfvt
nftMint: C9pVfJRZR1NspdRAyPkow553aynNDpLnFYqBWZD5mWT9
nftMint: EYKc62HiRwKADAYe5fSTXXFfU5B5XCLg8nSsJACMEcGj
nftMint: 2U3pGSVSpVUYcaz3bu4ZMaDpHQjpXLVHnx6FbzzS5TJa
nftMint: EvT3voPJNZTTBTYvQYFmgboj9JxEnJiQ5zxr45tX5MKY
nftMint: 5H87TEVxeAK5kYiDP9UCzF3kCQ8L6rx263gqWBy4B1An
```

The results can be found in `mint/mints.json` as well.

### Display NFTs in Phantom

Config `proxyman` for redirecting the http requests that are sent to testnet. This is a hack for customizing Phantom RPC endpoint.

Open `proxyman` and press `option` + `command` + `r` to set the Map Remote rules:

![](https://hackmd.io/_uploads/HyYO-7kiK.png)

Here, we redirect every requests to `https://rpc-mainnet-fork.dappio.xyz`, which is a `solana-mf` RPC operated by Dappio.

> Note: make sure you are not running any VPN software other than Proxyman in order to make the mapping work.

Change the network to `testnet` in Phantom:

![](https://hackmd.io/_uploads/HkMFGmksK.png)

Now, the NFTs should all be displaying:

![](https://hackmd.io/_uploads/HyGizQJjK.png)

That's it!

<!-- ## More Advanced Topics

- Candy Machine
 -->

## Reference

### General

- https://github.com/ilmoi/awesome-solana-nfts
- https://hackmd.io/@levicook/HJcDneEWF

### Metaplex

- https://medium.com/metaplex/metaplex-metadata-standard-45af3d04b541
- https://medium.com/coinmonks/structure-of-metaplex-nft-c6ef4834a803
- https://github.com/metaplex-foundation/metaplex-program-library/tree/master/token-metadata

### Arweave

- https://pencilflip.medium.com/how-to-use-arweave-to-store-and-access-nft-metadata-823552293f62
- https://pencilflip.medium.com/how-to-use-arweave-to-store-and-access-nft-metadata-part-2-21cb87f4091e

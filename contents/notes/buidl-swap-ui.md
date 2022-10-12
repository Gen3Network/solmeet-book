---
tags: notes
---

# Let’s BUIDL a Swap UI on Solana in 2 Hours

**Authors:** [@SaiyanBs](https://twitter.com/SaiyanBs), [@ironaddicteddog](https://twitter.com/ironaddicteddog)

***[Updated at 2022.3.31]***

> **See the example repo [here](https://github.com/DappioWonderland/swap-ui-example)**

> **See the demo DApp [here](https://swap-ui-example.dappio.xyz)**

## TL; DR

- Use Next.js + React.js + @solana/web3.js
- Raydium AMM swap
- Jupiter SDK swap

## Introduction

- What is Solana
    - Solana is a fast, low cost, decentralized blockchain with thousands of projects spanning DeFi, NFTs, Web3 and more.
- What is Raydium
    - Raydium is an Automated Market Maker (AMM) and liquidity provider built on the Solana blockchain.
- What is Jupiter
    - Jupiter is the key swap aggregator for Solana, offering the best route discovery between any token pair.

## Overview

### What does web3.js do?

- web3.js library
- Solana tx
- Solana ix

### How to find the program interface?

## Structure

```
├── 📂 pages
│   │
│   ├── 📂 api
│   │
│   ├── 📄 _app.tsx
│   │
│   ├── 📄 index.tsx
│   │
│   ├── 📄 jupiter.tsx
│   │
│   └── 📄 raydium.tsx
│
└── 📂 views
│   │
│   ├── 📂 commons
│   │
│   ├── 📂 jupiter
│   │
│   └── 📂 raydium
│
├── 📂 utils
│
├── 📂 styles
│
├── 📂 chakra
│   │
│   └── 📄 style.js
│
├── 📂 public
│
│── 📄 next.config.js
│
└── ...

```

## Setup

First, let's start a brand new next.js project:

```bash
$ npx create-next-app@latest solmeet-4-swap-ui --typescript
```

Remove `package-lock.json` since we will use `yarn` through this entire tutorial:

```bash
$ rm package-lock.json
```

### Install Dependencies

Next, let's install all the dependencies. This includes:

- Solana wallet adapter
- Solana web3.js
- Solana SPL token
- Serum
- Sass
- Jupiter SDK
- Next.js config plugins
- Chakra (UI lib)
- Lodash

Let's update `package.json` directly:

```json=
{
  "name": "solmeet-4-swap-ui",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "@chakra-ui/icons": "^1.1.1",
    "@chakra-ui/react": "^1.7.4",
    "@emotion/react": "^11",
    "@emotion/styled": "^11",
    "@jup-ag/react-hook": "^1.0.0-beta.2",
    "@project-serum/borsh": "^0.2.3",
    "@project-serum/serum": "^0.13.61",
    "@solana/spl-token-registry": "^0.2.1733",
    "@solana/wallet-adapter-base": "^0.9.2",
    "@solana/wallet-adapter-react": "^0.15.2",
    "@solana/wallet-adapter-react-ui": "^0.9.4",
    "@solana/wallet-adapter-wallets": "^0.14.2",
    "@solana/web3.js": "^1.32.0",
    "framer-motion": "^5",
    "lodash-es": "^4.17.21",
    "next": "12.0.8",
    "next-compose-plugins": "^2.2.1",
    "next-transpile-modules": "^9.0.0",
    "react": "17.0.2",
    "react-dom": "17.0.2",
    "sass": "^1.49.0"
  },
  "devDependencies": {
    "@types/lodash-es": "^4.17.5",
    "@types/node": "17.0.10",
    "@types/react": "17.0.38",
    "eslint": "8.7.0",
    "eslint-config-next": "12.0.8",
    "typescript": "4.5.5"
  },
  "resolutions": {
    "@solana/buffer-layout": "^3.0.0"
  }
}
```

Then run the installation:

```
$ yarn
...
```

> Note: make sure the version of `buffer-layout` is locked at `^3.0.0`

<!-- 
```bash
$ yarn add @solana/wallet-adapter-base @solana/wallet-adapter-react @solana/wallet-adapter-react-ui @solana/wallet-adapter-wallets @solana/web3.js @solana/spl-token-registry
```

```bash
$ yarn add @project-serum/borsh @project-serum/serum 
```

```bash
$ yarn add sass
```

```bash
$ yarn add @jup-ag/react-hook@1.0.0-beta.2
```

```bash
$ yarn add next-compose-plugins next-transpile-modules
```

```bash
$ yarn add @chakra-ui/icons @chakra-ui/react @emotion/react@^11 @emotion/styled@^11 framer-motion@^5
```

```bash
$ yarn add lodash-es
$ yarn add -D @types/lodash-es
```
 -->

### Scaffold

Populates folders and files for later update:

```bash
$ mkdir utils && touch utils/{ids.ts,layouts.ts,liquidity.ts,pools.ts,safe-math.ts,swap.ts,tokenList.ts,tokens.ts,web3.ts}
$ mkdir views && mkdir views/{commons,jupiter,raydium}
$ touch views/commons/{Navigator.tsx,WalletProvider.tsx,SplTokenList.tsx,Notify.tsx} && touch views/jupiter/{FeeInfo.tsx,JupiterForm.tsx,JupiterProvider.tsx} && touch views/raydium/{index.tsx,SlippageSetting.tsx,SwapOperateContainer.tsx,TokenList.tsx,TokenSelect.tsx,TitleRow.tsx}
$ touch styles/{swap.module.sass,color.module.sass,navigator.module.sass,jupiter.module.sass}
$ touch pages/{index.tsx,jupiter.tsx,raydium.tsx}
$ mkdir chakra && touch chakra/style.js
```

### Add Common Components

There are 4 common components:
- `Navigator`
- `Notify`
- `SplTokenList`
- `WalletProvider`

#### `Navigator`

Add the following code in `./views/commons/Navigator.tsx`:

```typescript=
import { FunctionComponent } from "react";
import Link from "next/link";
import {
  WalletModalProvider,
  WalletDisconnectButton,
  WalletMultiButton
} from "@solana/wallet-adapter-react-ui";
import { useWallet } from "@solana/wallet-adapter-react";
import style from "../../styles/navigator.module.sass";

const Navigator: FunctionComponent = () => {
  const wallet = useWallet();
  return (
    <div className={style.sidebar}>
      <div className={style.routesBlock}>
        <Link href="/" passHref>
          <a href="https://ibb.co/yP2vCNL">
            <img
              src="https://i.ibb.co/g9Yq8rs/logo-v4-horizontal-transparent.png"
              alt="logo-v4-horizontal-transparent"
              className={style.dappioLogo}
            />
          </a>
        </Link>
        <Link href="/jupiter">
          <a className={style.route}>Jupiter</a>
        </Link>
        <Link href="/raydium">
          <a className={style.route}>Raydium</a>
        </Link>
      </div>
      <WalletModalProvider>
        {wallet.connected ? <WalletDisconnectButton /> : <WalletMultiButton />}
      </WalletModalProvider>
    </div>
  );
};

export default Navigator;
```

#### `Notify`

Add the [following code](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/views/commons/Notify.tsx) in `./views/commons/Notify.tsx`:

```typescript=
import { FunctionComponent } from "react";
import {
  Alert,
  AlertIcon,
  AlertTitle,
  AlertDescription,
  AlertStatus
} from "@chakra-ui/react";
import style from "../../styles/swap.module.sass";

export interface INotify {
  status: AlertStatus;
  title: string;
  description: string;
  link?: string;
}
interface NotifyProps {
  message: {
    status: AlertStatus;
    title: string;
    description: string;
    link?: string;
  };
}

const Notify: FunctionComponent<NotifyProps> = props => {
  return (
    <Alert status={props.message.status} className={style.notifyContainer}>
      <div className={style.notifyTitleRow}>
        <AlertIcon boxSize="2rem" />
        <AlertTitle className={style.title}>{props.message.title}</AlertTitle>
      </div>
      <AlertDescription className={style.notifyDescription}>
        {props.message.description}
      </AlertDescription>
      {props.message.link ? (
        <a
          href={props.message.link}
          style={{ color: "#fbae21", textDecoration: "underline" }}
        >
          Check Explorer
        </a>
      ) : (
        ""
      )}
    </Alert>
  );
};

export default Notify;
```

#### `SplTokenList`

Add the [following code](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/views/commons/SplTokenList.tsx) in `./views/commons/SplTokenList.tsx`:

```typescript=
import { FunctionComponent } from "react";
import style from "../../styles/swap.module.sass";
import { TOKENS } from "../../utils/tokens";
import { ISplToken } from "../../utils/web3";

interface ISplTokenProps {
  splTokenData: ISplToken[];
}

interface SplTokenDisplayData {
  symbol: string;
  mint: string;
  pubkey: string;
  amount: number;
}

const SplTokenList: FunctionComponent<ISplTokenProps> = (
  props
): JSX.Element => {
  let tokenList: SplTokenDisplayData[] = [];
  if (props.splTokenData.length === 0) {
    return <></>;
  }

  for (const [_, value] of Object.entries(TOKENS)) {
    let spl: ISplToken | undefined = props.splTokenData.find(
      (t: ISplToken) => t.parsedInfo.mint === value.mintAddress
    );
    if (spl) {
      let token = {} as SplTokenDisplayData;
      token["symbol"] = value.symbol;
      token["mint"] = spl?.parsedInfo.mint;
      token["pubkey"] = spl?.pubkey;
      token["amount"] = spl?.amount;
      tokenList.push(token);
    }
  }

  let tokens = tokenList.map((item: SplTokenDisplayData) => {
    return (
      <div key={item.mint} className={style.splTokenItem}>
        <div>
          <span style={{ marginRight: "1rem", fontWeight: "600" }}>
            {item.symbol}
          </span>
          <span>- {item.amount}</span>
        </div>
        <div style={{ opacity: ".25" }}>
          <div>Mint: {item.mint}</div>
          <div>Pubkey: {item.pubkey}</div>
        </div>
      </div>
    );
  });

  return (
    <div className={style.splTokenContainer}>
      <div className={style.splTokenListTitle}>Your Tokens</div>
      {tokens}
    </div>
  );
};

export default SplTokenList;
```

#### `WalletProvider`

Add the [following code](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/views/commons/WalletProvider.tsx) in `./views/commons/WalletProvider.tsx`:

```typescript=
import React, { FunctionComponent, useMemo } from "react";
import {
  ConnectionProvider,
  WalletProvider
} from "@solana/wallet-adapter-react";
import { WalletAdapterNetwork } from "@solana/wallet-adapter-base";
import {
  LedgerWalletAdapter,
  PhantomWalletAdapter,
  SlopeWalletAdapter,
  SolflareWalletAdapter,
  SolletExtensionWalletAdapter,
  SolletWalletAdapter,
  TorusWalletAdapter
} from "@solana/wallet-adapter-wallets";
import { clusterApiUrl } from "@solana/web3.js";

// Default styles that can be overridden by your app
require("@solana/wallet-adapter-react-ui/styles.css");

export const Wallet: FunctionComponent = props => {
  // // The network can be set to 'devnet', 'testnet', or 'mainnet-beta'.
  const network = WalletAdapterNetwork.Mainnet;

  // // You can also provide a custom RPC endpoint.
  const endpoint = "https://rpc-mainnet-fork.dappio.xyz";

  // @solana/wallet-adapter-wallets includes all the adapters but supports tree shaking and lazy loading --
  // Only the wallets you configure here will be compiled into your application, and only the dependencies
  // of wallets that your users connect to will be loaded.
  const wallets = useMemo(
    () => [
      new PhantomWalletAdapter(),
      new SlopeWalletAdapter(),
      new SolflareWalletAdapter(),
      new TorusWalletAdapter(),
      new LedgerWalletAdapter(),
      new SolletWalletAdapter({ network }),
      new SolletExtensionWalletAdapter({ network })
    ],
    [network]
  );

  return (
    <ConnectionProvider endpoint={endpoint}>
      <WalletProvider wallets={wallets} autoConnect>
        {props.children}
      </WalletProvider>
    </ConnectionProvider>
  );
};
```

### Add Pages for `Raydium` and `Jupiter`

Add the following code in `./pages/raydium.tsx`:

```typescript=
import { FunctionComponent } from "react";

const RaydiumPage: FunctionComponent = () => {
  return <div>This is Raydium Page</div>;
};

export default RaydiumPage;
```

Add the following code in `./pages/jupiter.tsx`:

```typescript=
import { FunctionComponent } from "react";

const JupiterPage: FunctionComponent = () => {
  return <div>This is Jupiter Page</div>;
};

export default JupiterPage;
```

### Update Styles

Theere are 3 style sheets to be updated:
- `globals.css`
- `navigator.module.sass`
- `color.module.sass`

#### `globals.css`

Add the following code in `./styles/globals.css`:

```css=
html,
body {
  font-size: 10px;
  background-color: rgb(19, 27, 51);
  color: #eee
}

.wallet-adapter-modal-list-more {
  color: #eee
}
.wallet-adapter-button-trigger {
  background-color: #fbae21 !important;
  color: black !important
}
```

#### `navigator.module.sass`

Add the following code in `./styles/navigator.module.sass`:

```sass=
@import './color.module.sass'

.dappioLogo
  flex: 2
  text-align: center
  width: 12rem
  margin-right: 10rem
  cursor: pointer
.sidebar
  display: flex
  align-items: center
  font-size: 2rem
  height: 7rem
  border-bottom: 1px solid rgb(29, 40, 76)
  background-color: $main_blue
  padding: 0 4rem
  justify-content: space-between
  letter-spacing: .1rem
  font-weight: 500
.routesBlock
  display: flex
  align-items: center
  justify-content: space-around
  color: $white
  font-size: 1.5rem
.route
  margin-right: 5rem
```

#### `color.module.sass`

Add the following code in `./styles/color.module.sass`:

```sass=
$white: #eee
$main_blue: rgb(19, 27, 51)
$swap_card_bgc: #131a35
$coin_select_block_bgc: #000829
$placeholder_grey: #f1f1f2
$swap_btn_border_color: #5ac4be
$token_list_bgc: #1c274f
$slippage_setting_warning_red: #f5222d
```

### Update `app`

Replace `pages/_app.tsx` with following code:

```typescript=
import "../styles/globals.css";
import type { AppProps } from "next/app";
import { Wallet } from "../views/commons/WalletProvider";
import Navigator from "../views/commons/Navigator";

function SwapUI({ Component, pageProps }: AppProps) {
  return (
    <>
      <Wallet>
        <Navigator />
        <Component {...pageProps} />
      </Wallet>
    </>
  );
}

export default SwapUI;
``` 

Start the dev server. For now you should see jupiter and raydium page with only plain text and one wallet connecting button:

```bash
$ yarn dev
```

## Part 1: Build a Swap on Raydium

### What we need to implement Raydium swap?

1. Token list
2. Slippage setting
3. Price out
4. Amm pools info
5. Interact with on-chain program

### Add Raydium Utils

Let's update each component one by one:
- [ids.ts](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/utils/ids.ts)
- [layouts.ts](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/utils/layouts.ts)
- [liquidity.ts](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/utils/liquidity.ts)
- [pools.ts](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/utils/pools.ts)
- [safe-math.ts](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/utils/safe-math.ts)
- [swap.ts](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/utils/swap.ts)
- [tokenList.ts](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/utils/tokenList.ts)
- [tokens.ts](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/utils/tokens.ts)
- [web3.ts](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/utils/web3.ts)

### Add Components

We will update the following components:
- `SlippageSetting`
- `SwapOperateContainer`
- `TitleRow`
- `TokenList`
- `TokenSelect`
- `index`

#### `TitleRow.tsx`

Add the [following code](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/views/raydium/TitleRow.tsx) in `./views/raydium/TitleRow.tsx`:

```typescript=
import style from "../../styles/swap.module.sass";
import {
  Tooltip,
  Popover,
  PopoverTrigger,
  PopoverContent,
  PopoverBody,
  PopoverArrow
} from "@chakra-ui/react";
import { SettingsIcon, InfoOutlineIcon } from "@chakra-ui/icons";
import { useState, useEffect, FunctionComponent } from "react";
import { TokenData, ITokenInfo } from ".";

interface ITitleProps {
  toggleSlippageSetting: Function;
  fromData: TokenData;
  toData: TokenData;
  updateSwapOutAmount: Function;
}

interface IAddressInfoProps {
  type: string;
}

const TitleRow: FunctionComponent<ITitleProps> = (props): JSX.Element => {
  const [second, setSecond] = useState<number>(0);
  const [percentage, setPercentage] = useState<number>(0);

  useEffect(() => {
    let id = setInterval(() => {
      setSecond(second + 1);
      setPercentage((second * 100) / 60);
      if (second === 60) {
        setSecond(0);
        props.updateSwapOutAmount();
      }
    }, 1000);
    return () => clearInterval(id);
  });

  const AddressInfo: FunctionComponent<IAddressInfoProps> = (
    addressProps
  ): JSX.Element => {
    let fromToData = {} as ITokenInfo;
    if (addressProps.type === "From") {
      fromToData = props.fromData.tokenInfo;
    } else {
      fromToData = props.toData.tokenInfo;
    }

    return (
      <>
        <span className={style.symbol}>{fromToData?.symbol}</span>
        <span className={style.address}>
          <span>{fromToData?.mintAddress.substring(0, 14)}</span>
          <span>{fromToData?.mintAddress ? "..." : ""}</span>
          {fromToData?.mintAddress.substr(-14)}
        </span>
      </>
    );
  };

  return (
    <div className={style.titleContainer}>
      <div className={style.title}>Swap</div>
      <div className={style.iconContainer}>
        <Tooltip
          hasArrow
          label={`Displayed data will auto-refresh after ${
            60 - second
          } seconds. Click this circle to update manually.`}
          color="white"
          bg="brand.100"
          padding="3"
        >
          <svg
            viewBox="0 0 36 36"
            className={`${style.percentageCircle} ${style.icon}`}
          >
            <path
              className={style.circleBg}
              d="M18 2.0845
              a 15.9155 15.9155 0 0 1 0 31.831
              a 15.9155 15.9155 0 0 1 0 -31.831"
            />
            <path
              d="M18 2.0845
              a 15.9155 15.9155 0 0 1 0 31.831
              a 15.9155 15.9155 0 0 1 0 -31.831"
              fill="none"
              stroke="rgb(20, 120, 227)"
              strokeWidth="3"
              // @ts-ignore
              strokeDasharray={[percentage, 100]}
            />
          </svg>
        </Tooltip>
        <Popover trigger="hover">
          <PopoverTrigger>
            <div className={style.icon}>
              <InfoOutlineIcon w={18} h={18} />
            </div>
          </PopoverTrigger>
          <PopoverContent
            color="white"
            bg="brand.100"
            border="none"
            w="auto"
            className={style.popover}
          >
            <PopoverArrow bg="brand.100" className={style.popover} />
            <PopoverBody>
              <div className={style.selectTokenAddressTitle}>
                Program Addresses (DO NOT DEPOSIT)
              </div>
              <div className={style.selectTokenAddress}>
                {props.fromData.tokenInfo?.symbol ? (
                  <AddressInfo type="From" />
                ) : (
                  ""
                )}
              </div>
              <div className={style.selectTokenAddress}>
                {props.toData.tokenInfo?.symbol ? (
                  <AddressInfo type="To" />
                ) : (
                  ""
                )}
              </div>
            </PopoverBody>
          </PopoverContent>
        </Popover>
        <div
          className={style.icon}
          onClick={() => props.toggleSlippageSetting()}
        >
          <SettingsIcon w={18} h={18} />
        </div>
      </div>
    </div>
  );
};

export default TitleRow;
```

#### `TokenList.tsx`

Add the [following code](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/views/raydium/TokenList.tsx) in `./views/raydium/TokenList.tsx`:

```typescript=
import { FunctionComponent, useEffect, useRef, useState } from "react";
import { CloseIcon } from "@chakra-ui/icons";
import SPLTokenRegistrySource from "../../utils/tokenList";
import { TOKENS } from "../../utils/tokens";
import { ITokenInfo } from ".";
import style from "../../styles/swap.module.sass";

interface TokenListProps {
  showTokenList: boolean;
  toggleTokenList: (event?: React.MouseEvent<HTMLDivElement>) => void;
  getTokenInfo: Function;
}

const TokenList: FunctionComponent<TokenListProps> = props => {
  const [initialList, setList] = useState<ITokenInfo[]>([]);
  const [searchedList, setSearchList] = useState<ITokenInfo[]>([]);
  const searchRef = useRef<any>();

  useEffect(() => {
    SPLTokenRegistrySource().then((res: any) => {
      let list: ITokenInfo[] = [];
      res.map((item: any) => {
        let token = {} as ITokenInfo;
        if (
          TOKENS[item.symbol] &&
          !list.find(
            (t: ITokenInfo) => t.mintAddress === TOKENS[item.symbol].mintAddress
          )
        ) {
          token = TOKENS[item.symbol];
          token["logoURI"] = item.logoURI;
          list.push(token);
        }
      });
      setList(() => list);
      props.getTokenInfo(
        list.find((item: ITokenInfo) => item.symbol === "SOL")
      );
    });
  }, []);

  useEffect(() => {
    setSearchList(() => initialList);
  }, [initialList]);

  const setTokenInfo = (item: ITokenInfo) => {
    props.getTokenInfo(item);
    props.toggleTokenList();
  };

  useEffect(() => {
    if (!props.showTokenList) {
      setSearchList(initialList);
      searchRef.current.value = "";
    }
  }, [props.showTokenList]);

  const listItems = (data: ITokenInfo[]) => {
    return data.map((item: ITokenInfo) => {
      return (
        <div
          className={style.tokenRow}
          key={item.mintAddress}
          onClick={() => setTokenInfo(item)}
        >
          <img src={item.logoURI} alt="" className={style.tokenLogo} />
          <div>{item.symbol}</div>
        </div>
      );
    });
  };

  const searchToken = (e: any) => {
    let key = e.target.value.toUpperCase();
    let newList: ITokenInfo[] = [];
    initialList.map((item: ITokenInfo) => {
      if (item.symbol.includes(key)) {
        newList.push(item);
      }
    });
    setSearchList(() => newList);
  };

  let tokeListComponentStyle;
  if (!props.showTokenList) {
    tokeListComponentStyle = {
      display: "none"
    };
  } else {
    tokeListComponentStyle = {
      display: "block"
    };
  }

  return (
    <div className={style.tokeListComponent} style={tokeListComponentStyle}>
      <div className={style.tokeListContainer}>
        <div className={style.header}>
          <div>Select a token</div>
          <div className={style.closeIcon} onClick={props.toggleTokenList}>
            <CloseIcon w={5} h={5} />
          </div>
        </div>
        <div className={style.inputBlock}>
          <input
            type="text"
            placeholder="Search name or mint address"
            ref={searchRef}
            className={style.searchTokenInput}
            onChange={searchToken}
          />
          <div className={style.tokenListTitleRow}>
            <div>Token name</div>
          </div>
        </div>
        <div className={style.list}>{listItems(searchedList)}</div>
        <div className={style.tokenListSetting}>View Token List</div>
      </div>
    </div>
  );
};

export default TokenList;
```

#### `SlippageSetting.tsx`

Add the [following code](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/views/raydium/SlippageSetting.tsx) in `./views/raydium/SlippageSetting.tsx`:

```typescript=
import { useState, useEffect, FunctionComponent } from "react";
import { CloseIcon } from "@chakra-ui/icons";
import style from "../../styles/swap.module.sass";

interface SlippageSettingProps {
  showSlippageSetting: boolean;
  toggleSlippageSetting: Function;
  getSlippageValue: Function;
  slippageValue: number;
}

const SlippageSetting: FunctionComponent<SlippageSettingProps> = props => {
  const rate = [0.1, 0.5, 1];
  const [warningText, setWarningText] = useState("");

  const setSlippageBtn = (item: number) => {
    props.getSlippageValue(item);
  };

  useEffect(() => {
    Options();

    if (props.slippageValue < 0) {
      setWarningText("Please enter a valid slippage percentage");
    } else if (props.slippageValue < 1) {
      setWarningText("Your transaction may fail");
    } else {
      setWarningText("");
    }
  }, [props.slippageValue]);

  const Options = (): JSX.Element => {
    return (
      <>
        {rate.map(item => {
          return (
            <button
              className={`${style.optionBtn} ${
                item === props.slippageValue
                  ? style.selectedSlippageRateBtn
                  : ""
              }`}
              key={item}
              onClick={() => setSlippageBtn(item)}
            >
              {item}%
            </button>
          );
        })}
      </>
    );
  };

  const updateInputRate = (e: React.FormEvent<HTMLInputElement>) => {
    props.getSlippageValue(e.currentTarget.value);
  };

  const close = () => {
    if (props.slippageValue < 0) {
      return;
    }
    props.toggleSlippageSetting();
  };

  if (!props.showSlippageSetting) {
    return null;
  }

  return (
    <div className={style.slippageSettingComponent}>
      <div className={style.slippageSettingContainer}>
        <div className={style.header}>
          <div>Setting</div>
          <div className={style.closeIcon} onClick={close}>
            <CloseIcon w={5} h={5} />
          </div>
        </div>
        <div className={style.settingSelectBlock}>
          <div className={style.title}>Slippage tolerance</div>
          <div className={style.optionsBlock}>
            <Options />
            <button className={`${style.optionBtn} ${style.inputBtn}`}>
              <input
                type="number"
                placeholder="0%"
                className={style.input}
                value={props.slippageValue}
                onChange={updateInputRate}
              />
              %
            </button>
          </div>
          <div className={style.warning}>{warningText}</div>
        </div>
      </div>
    </div>
  );
};

export default SlippageSetting;
```

#### `TokenSelect.tsx`

Add the [following code](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/views/raydium/TokenSelect.tsx) in `./views/raydium/TokenSelect.tsx`:

```typescript=
import { FunctionComponent, useEffect, useState } from "react";
import { ArrowDownIcon } from "@chakra-ui/icons";
import { useWallet } from "@solana/wallet-adapter-react";
import { TokenData } from "./index";
import { ISplToken } from "../../utils/web3";
import style from "../../styles/swap.module.sass";

interface TokenSelectProps {
  type: string;
  toggleTokenList: Function;
  tokenData: TokenData;
  updateAmount: Function;
  wallet: Object;
  splTokenData: ISplToken[];
}

export interface IUpdateAmountData {
  type: string;
  amount: number;
}

interface SelectTokenProps {
  propsData: {
    tokenData: TokenData;
  };
}

const TokenSelect: FunctionComponent<TokenSelectProps> = props => {
  let wallet = useWallet();
  const [tokenBalance, setTokenBalance] = useState<number | null>(null);

  const updateAmount = (e: any) => {
    e.preventDefault();

    const amountData: IUpdateAmountData = {
      amount: e.target.value,
      type: props.type
    };
    props.updateAmount(amountData);
  };

  const selectToken = () => {
    props.toggleTokenList(props.type);
  };

  useEffect(() => {
    const getTokenBalance = () => {
      let data: ISplToken | undefined = props.splTokenData.find(
        (t: ISplToken) =>
          t.parsedInfo.mint === props.tokenData.tokenInfo?.mintAddress
      );

      if (data) {
        //@ts-ignore
        setTokenBalance(data.amount);
      }
    };
    getTokenBalance();
  }, [props.splTokenData]);

  const SelectTokenBtn: FunctionComponent<
    SelectTokenProps
  > = selectTokenProps => {
    if (selectTokenProps.propsData.tokenData.tokenInfo?.symbol) {
      return (
        <>
          <img
            src={selectTokenProps.propsData.tokenData.tokenInfo?.logoURI}
            alt="logo"
            className={style.img}
          />
          <div className={style.coinNameBlock}>
            <span className={style.coinName}>
              {selectTokenProps.propsData.tokenData.tokenInfo?.symbol}
            </span>
            <ArrowDownIcon w={5} h={5} />
          </div>
        </>
      );
    }
    return (
      <>
        <span>Select a token</span>
        <ArrowDownIcon w={5} h={5} />
      </>
    );
  };

  return (
    <div className={style.coinSelect}>
      <div className={style.noteText}>
        <div>
          {props.type === "To" ? `${props.type} (Estimate)` : props.type}
        </div>
        <div>
          {wallet.connected && tokenBalance
            ? `Balance: ${tokenBalance.toFixed(4)}`
            : ""}
        </div>
      </div>
      <div className={style.coinAmountRow}>
        {props.type !== "From" ? (
          <div className={style.input}>
            {props.tokenData.amount ? props.tokenData.amount : "-"}
          </div>
        ) : (
          <input
            type="number"
            className={style.input}
            placeholder="0.00"
            onChange={updateAmount}
            disabled={props.type !== "From"}
          />
        )}

        <div className={style.selectTokenBtn} onClick={selectToken}>
          <SelectTokenBtn propsData={props} />
        </div>
      </div>
    </div>
  );
};

export default TokenSelect;
```

#### `SwapOperateContainer.tsx`

Add the [following code](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/views/raydium/SwapOperateContainer.tsx) in `./views/raydium/SwapOperateContainer.tsx`:

```typescript=
import { FunctionComponent } from "react";
import { ArrowUpDownIcon, QuestionOutlineIcon } from "@chakra-ui/icons";
import { Tooltip } from "@chakra-ui/react";
import { useWallet } from "@solana/wallet-adapter-react";
import {
  WalletModalProvider,
  WalletMultiButton
} from "@solana/wallet-adapter-react-ui";
import { TokenData } from ".";
import TokenSelect from "./TokenSelect";
import { ISplToken } from "../../utils/web3";
import style from "../../styles/swap.module.sass";

interface SwapOperateContainerProps {
  toggleTokenList: Function;
  fromData: TokenData;
  toData: TokenData;
  updateAmount: Function;
  switchFromAndTo: (event?: React.MouseEvent<HTMLDivElement>) => void;
  slippageValue: number;
  sendSwapTransaction: (event?: React.MouseEvent<HTMLButtonElement>) => void;
  splTokenData: ISplToken[];
}

interface SwapDetailProps {
  title: string;
  tooltipContent: string;
  value: string;
}

const SwapOperateContainer: FunctionComponent<
  SwapOperateContainerProps
> = props => {
  let wallet = useWallet();
  const SwapBtn = (swapProps: any) => {
    if (wallet.connected) {
      if (
        !swapProps.props.fromData.tokenInfo?.symbol ||
        !swapProps.props.toData.tokenInfo?.symbol
      ) {
        return (
          <button
            className={`${style.operateBtn} ${style.disabledBtn}`}
            disabled
          >
            Select a token
          </button>
        );
      }
      if (
        swapProps.props.fromData.tokenInfo?.symbol &&
        swapProps.props.toData.tokenInfo?.symbol
      ) {
        if (
          !swapProps.props.fromData.amount ||
          !swapProps.props.toData.amount
        ) {
          return (
            <button
              className={`${style.operateBtn} ${style.disabledBtn}`}
              disabled
            >
              Enter an amount
            </button>
          );
        }
      }

      return (
        <button
          className={style.operateBtn}
          onClick={props.sendSwapTransaction}
        >
          Swap
        </button>
      );
    } else {
      return (
        <div className={style.selectWallet}>
          <WalletModalProvider>
            <WalletMultiButton />
          </WalletModalProvider>
        </div>
      );
    }
  };

  const SwapDetailPreview: FunctionComponent<SwapDetailProps> = props => {
    return (
      <div className={style.slippageRow}>
        <div className={style.slippageTooltipBlock}>
          <div>{props.title}</div>
          <Tooltip
            hasArrow
            label={props.tooltipContent}
            color="white"
            bg="brand.100"
            padding="3"
          >
            <QuestionOutlineIcon
              w={5}
              h={5}
              className={`${style.icon} ${style.icon}`}
            />
          </Tooltip>
        </div>
        <div>{props.value}</div>
      </div>
    );
  };

  const SwapDetailPreviewList = (): JSX.Element => {
    return (
      <>
        <SwapDetailPreview
          title="Swapping Through"
          tooltipContent="This venue gave the best price for your trade"
          value={`${props.fromData.tokenInfo.symbol} > ${props.toData.tokenInfo.symbol}`}
        />
      </>
    );
  };

  return (
    <div className={style.swapCard}>
      <div className={style.cardBody}>
        <TokenSelect
          type="From"
          toggleTokenList={props.toggleTokenList}
          tokenData={props.fromData}
          updateAmount={props.updateAmount}
          wallet={wallet}
          splTokenData={props.splTokenData}
        />
        <div
          className={`${style.switchIcon} ${style.icon}`}
          onClick={props.switchFromAndTo}
        >
          <ArrowUpDownIcon w={5} h={5} />
        </div>
        <TokenSelect
          type="To"
          toggleTokenList={props.toggleTokenList}
          tokenData={props.toData}
          updateAmount={props.updateAmount}
          wallet={wallet}
          splTokenData={props.splTokenData}
        />
        <div className={style.slippageRow}>
          <div className={style.slippageTooltipBlock}>
            <div>Slippage Tolerance </div>
            <Tooltip
              hasArrow
              label="The maximum difference between your estimated price and execution price."
              color="white"
              bg="brand.100"
              padding="3"
            >
              <QuestionOutlineIcon
                w={5}
                h={5}
                className={`${style.icon} ${style.icon}`}
              />
            </Tooltip>
          </div>
          <div>{props.slippageValue}%</div>
        </div>
        {props.fromData.amount! > 0 &&
        props.fromData.tokenInfo.symbol &&
        props.toData.amount! > 0 &&
        props.toData.tokenInfo.symbol ? (
          <SwapDetailPreviewList />
        ) : (
          ""
        )}
        <SwapBtn props={props} />
      </div>
    </div>
  );
};

export default SwapOperateContainer;
```

#### `index`

Add the [following code](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/views/raydium/index.tsx) in `./views/raydium/index.tsx`:

```typescript=
import { useState, useEffect, FunctionComponent } from "react";
import TokenList from "./TokenList";
import TitleRow from "./TitleRow";
import SlippageSetting from "./SlippageSetting";
import SwapOperateContainer from "./SwapOperateContainer";
import { Connection } from "@solana/web3.js";
import { Spinner } from "@chakra-ui/react";
import { useWallet, WalletContextState } from "@solana/wallet-adapter-react";
import { getPoolByTokenMintAddresses } from "../../utils/pools";
import { swap, getSwapOutAmount, setupPools } from "../../utils/swap";
import { getSPLTokenData } from "../../utils/web3";
import Notify from "../commons/Notify";
import { INotify } from "../commons/Notify";
import SplTokenList from "../commons/SplTokenList";
import { ISplToken } from "../../utils/web3";
import { IUpdateAmountData } from "./TokenSelect";
import style from "../../styles/swap.module.sass";

export interface ITokenInfo {
  symbol: string;
  mintAddress: string;
  logoURI: string;
}
export interface TokenData {
  amount: number | null;
  tokenInfo: ITokenInfo;
}

const SwapPage: FunctionComponent = () => {
  const [showTokenList, setShowTokenList] = useState(false);
  const [showSlippageSetting, setShowSlippageSetting] = useState(false);
  const [selectType, setSelectType] = useState<string>("From");
  const [fromData, setFromData] = useState<TokenData>({} as TokenData);
  const [toData, setToData] = useState<TokenData>({} as TokenData);
  const [slippageValue, setSlippageValue] = useState(1);
  const [splTokenData, setSplTokenData] = useState<ISplToken[]>([]);
  const [liquidityPools, setLiquidityPools] = useState<any>("");
  const [isLoading, setIsLoading] = useState<boolean>(false);
  const [notify, setNotify] = useState<INotify>({
    status: "info",
    title: "",
    description: "",
    link: ""
  });
  const [showNotify, toggleNotify] = useState<Boolean>(false);

  let wallet: WalletContextState = useWallet();
  const connection = new Connection("https://rpc-mainnet-fork.dappio.xyz", {
    wsEndpoint: "wss://rpc-mainnet-fork.dappio.xyz/ws",
    commitment: "processed"
  });

  useEffect(() => {
    setIsLoading(true);
    setupPools(connection).then(data => {
      setLiquidityPools(data);
      setIsLoading(false);
    });
    return () => {
      setLiquidityPools("");
    };
  }, []);

  useEffect(() => {
    if (wallet.connected) {
      getSPLTokenData(wallet, connection).then((tokenList: ISplToken[]) => {
        if (tokenList) {
          setSplTokenData(() => tokenList.filter(t => t !== undefined));
        }
      });
    }
  }, [wallet.connected]);

  const updateAmount = (e: IUpdateAmountData) => {
    if (e.type === "From") {
      setFromData((old: TokenData) => ({
        ...old,
        amount: e.amount
      }));

      if (!e.amount) {
        setToData((old: TokenData) => ({
          ...old,
          amount: 0
        }));
      }
    }
  };

  const updateSwapOutAmount = () => {
    if (
      fromData.amount! > 0 &&
      fromData.tokenInfo?.symbol &&
      toData.tokenInfo?.symbol
    ) {
      let poolInfo = getPoolByTokenMintAddresses(
        fromData.tokenInfo.mintAddress,
        toData.tokenInfo.mintAddress
      );
      if (!poolInfo) {
        setNotify((old: INotify) => ({
          ...old,
          status: "error",
          title: "AMM error",
          description: "Current token pair pool not found"
        }));
        toggleNotify(true);
        return;
      }

      let parsedPoolsData = liquidityPools;
      let parsedPoolInfo = parsedPoolsData[poolInfo?.lp.mintAddress];

      // //@ts-ignore
      const { amountOutWithSlippage } = getSwapOutAmount(
        parsedPoolInfo,
        fromData.tokenInfo.mintAddress,
        toData.tokenInfo.mintAddress,
        fromData.amount!.toString(),
        slippageValue
      );

      setToData((old: TokenData) => ({
        ...old,
        amount: parseFloat(amountOutWithSlippage.fixed())
      }));
    }
  };

  useEffect(() => {
    updateSwapOutAmount();
  }, [fromData]);

  useEffect(() => {
    updateSwapOutAmount();
  }, [toData.tokenInfo?.symbol]);

  useEffect(() => {
    updateSwapOutAmount();
  }, [slippageValue]);

  const toggleTokenList = (e: any) => {
    setShowTokenList(() => !showTokenList);
    setSelectType(() => e);
  };

  const toggleSlippageSetting = () => {
    setShowSlippageSetting(() => !showSlippageSetting);
  };

  const getSlippageValue = (e: number) => {
    if (!e) {
      setSlippageValue(() => e);
    } else {
      setSlippageValue(() => e);
    }
  };

  const switchFromAndTo = () => {
    const fromToken = fromData.tokenInfo;
    const toToken = toData.tokenInfo;
    setFromData((old: TokenData) => ({
      ...old,
      tokenInfo: toToken,
      amount: null
    }));

    setToData((old: TokenData) => ({
      ...old,
      tokenInfo: fromToken,
      amount: null
    }));
  };

  const getTokenInfo = (e: any) => {
    if (selectType === "From") {
      if (toData.tokenInfo?.symbol === e?.symbol) {
        setToData((old: TokenData) => ({
          ...old,
          tokenInfo: {
            symbol: "",
            mintAddress: "",
            logoURI: ""
          }
        }));
      }

      setFromData((old: TokenData) => ({
        ...old,
        tokenInfo: e
      }));
    } else {
      if (fromData.tokenInfo?.symbol === e.symbol) {
        setFromData((old: TokenData) => ({
          ...old,
          tokenInfo: {
            symbol: "",
            mintAddress: "",
            logoURI: ""
          }
        }));
      }

      setToData((old: TokenData) => ({
        ...old,
        tokenInfo: e
      }));
    }
  };

  const sendSwapTransaction = async () => {
    let poolInfo = getPoolByTokenMintAddresses(
      fromData.tokenInfo.mintAddress,
      toData.tokenInfo.mintAddress
    );

    let fromTokenAccount: ISplToken | undefined | string = splTokenData.find(
      (token: ISplToken) =>
        token.parsedInfo.mint === fromData.tokenInfo.mintAddress
    );
    if (fromTokenAccount) {
      fromTokenAccount = fromTokenAccount.pubkey;
    } else {
      fromTokenAccount = "";
    }

    let toTokenAccount: ISplToken | undefined | string = splTokenData.find(
      (token: ISplToken) =>
        token.parsedInfo.mint === toData.tokenInfo.mintAddress
    );
    if (toTokenAccount) {
      toTokenAccount = toTokenAccount.pubkey;
    } else {
      toTokenAccount = "";
    }

    let wsol: ISplToken | undefined = splTokenData.find(
      (token: ISplToken) =>
        token.parsedInfo.mint === "So11111111111111111111111111111111111111112"
    );
    let wsolMint: string = "";
    if (wsol) {
      wsolMint = wsol.parsedInfo.mint;
    }

    if (poolInfo === undefined) {
      alert("Pool not exist");
      return;
    }

    swap(
      connection,
      wallet,
      poolInfo,
      fromData.tokenInfo.mintAddress,
      toData.tokenInfo.mintAddress,
      fromTokenAccount,
      toTokenAccount,
      fromData.amount!.toString(),
      toData.amount!.toString(),
      wsolMint
    ).then(async res => {
      toggleNotify(true);
      setNotify((old: INotify) => ({
        ...old,
        status: "success",
        title: "Transaction Send",
        description: "",
        link: `https://explorer.solana.com/address/${res}`
      }));

      let result = await connection.confirmTransaction(res);

      if (!result.value.err) {
        setNotify((old: INotify) => ({
          ...old,
          status: "success",
          title: "Transaction Success"
        }));
      } else {
        setNotify((old: INotify) => ({
          ...old,
          status: "success",
          title: "Fail",
          description: "Transaction fail, please check below link",
          link: `https://explorer.solana.com/address/${res}`
        }));
      }

      getSPLTokenData(wallet, connection).then((tokenList: ISplToken[]) => {
        if (tokenList) {
          setSplTokenData(() =>
            tokenList.filter((t: ISplToken) => t !== undefined)
          );
        }
      });
    });
  };

  useEffect(() => {
    const time = setTimeout(() => {
      toggleNotify(false);
    }, 8000);

    return () => clearTimeout(time);
  }, [notify]);

  useEffect(() => {
    if (wallet.connected) {
      setNotify((old: INotify) => ({
        ...old,
        status: "success",
        title: "Wallet connected",
        description: wallet.publicKey?.toBase58() as string
      }));
    } else {
      let description = wallet.publicKey?.toBase58();
      if (!description) {
        description = "Please try again";
      }
      setNotify((old: INotify) => ({
        ...old,
        status: "error",
        title: "Wallet disconnected",
        description: description as string
      }));
    }

    toggleNotify(true);
  }, [wallet.connected]);

  return (
    <div className={style.swapPage}>
      {isLoading ? (
        <div className={style.loading}>
          Loading raydium amm pool <Spinner />
        </div>
      ) : (
        ""
      )}
      <SplTokenList splTokenData={splTokenData} />
      <SlippageSetting
        showSlippageSetting={showSlippageSetting}
        toggleSlippageSetting={toggleSlippageSetting}
        getSlippageValue={getSlippageValue}
        slippageValue={slippageValue}
      />
      <TokenList
        showTokenList={showTokenList}
        toggleTokenList={toggleTokenList}
        getTokenInfo={getTokenInfo}
      />
      <div className={style.container}>
        {isLoading ? (
          ""
        ) : (
          <>
            <TitleRow
              toggleSlippageSetting={toggleSlippageSetting}
              fromData={fromData}
              toData={toData}
              updateSwapOutAmount={updateSwapOutAmount}
            />
            <SwapOperateContainer
              toggleTokenList={toggleTokenList}
              fromData={fromData}
              toData={toData}
              updateAmount={updateAmount}
              switchFromAndTo={switchFromAndTo}
              slippageValue={slippageValue}
              sendSwapTransaction={sendSwapTransaction}
              splTokenData={splTokenData}
            />
          </>
        )}
      </div>
      {showNotify ? <Notify message={notify} /> : null}
    </div>
  );
};

export default SwapPage;
```

### Update Style

Add code in`./styles/swap.module.sass` from [swap.module.sass](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/styles/swap.module.sass).

Add the following code in `./chakra/style.js`

```javascript=
import {
  extendTheme
} from "@chakra-ui/react"

const theme = extendTheme({
  colors: {
    brand: {
      100: "#1c274f"
    },
  },
})

export default theme
```

### Update Config

Replace `next.config.js` with following code:

```javascript=
/** @type {import('next').NextConfig} */
const withPlugins = require("next-compose-plugins");

/** eslint-disable @typescript-eslint/no-var-requires */
const withTM = require("next-transpile-modules")([
  "@solana/wallet-adapter-base",
  // Uncomment wallets you want to use
  // "@solana/wallet-adapter-bitpie",
  // "@solana/wallet-adapter-coin98",
  // "@solana/wallet-adapter-ledger",
  // "@solana/wallet-adapter-mathwallet",
  "@solana/wallet-adapter-phantom",
  "@solana/wallet-adapter-react",
  "@solana/wallet-adapter-solflare",
  "@solana/wallet-adapter-sollet",
  // "@solana/wallet-adapter-solong",
  // "@solana/wallet-adapter-torus",
  "@solana/wallet-adapter-wallets",
  // "@project-serum/sol-wallet-adapter",
  // "@solana/wallet-adapter-ant-design",
]);

const plugins = [
  [
    withTM,
    {
      webpack5: true,
      reactStrictMode: true,
    },
  ],
];

const nextConfig = {
  swcMinify: false,
  webpack: (config, {
    isServer
  }) => {
    if (!isServer) {
      config.resolve.fallback.fs = false;
    }
    return config;
  },
};

module.exports = withPlugins(plugins, nextConfig);
```

### Update `Raydium` Page

Finally, update `./pages/raydium.tsx`:

```typescript=
import { FunctionComponent } from "react";
import Swap from "../views/raydium/index";
import { ChakraProvider } from "@chakra-ui/react";
import theme from "../chakra/style";

const RaydiumPage: FunctionComponent = () => {
  return (
    <div>
      <ChakraProvider theme={theme}>
        <Swap />
      </ChakraProvider>
    </div>
  );
};

export default RaydiumPage;
```

Restart the dev server:

```
$ yarn dev
```

## Part 2: Build a Swap on Juipter Aggreggator

### How does Jupiter work?

### Add Components

We will update the following components:
- `FeeInfo`
- `JupiterForm`
- `JupiterProvider`

#### `JupiterProvider.tsx`

Add the [following code](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/views/jupiter/JupiterProvider.tsx) in `./views/jupiter/JupiterProvider.tsx`:

```typescript=
import { FunctionComponent } from "react";
import { JupiterProvider } from "@jup-ag/react-hook";
import { Connection } from "@solana/web3.js";
import { useWallet } from "@solana/wallet-adapter-react";
const connection = new Connection("https://rpc-mainnet-fork.dappio.xyz", {
  wsEndpoint: "wss://rpc-mainnet-fork.dappio.xyz/ws",
  commitment: "processed"
});

const Jupiter: FunctionComponent = ({ children }) => {
  const wallet = useWallet();
  return (
    <JupiterProvider
      connection={connection}
      cluster="mainnet-beta"
      userPublicKey={wallet.publicKey || undefined}
    >
      {children}
    </JupiterProvider>
  );
};

export default Jupiter;
```

#### `FeeInfo.tsx`

Add the [following code](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/views/jupiter/FeeInfo.tsx) in `./views/jupiter/FeeInfo.tsx`:

```typescript=
import React, { FunctionComponent, useEffect, useState } from "react";
import { RouteInfo, TransactionFeeInfo } from "@jup-ag/react-hook";

const FeeInfo: FunctionComponent<{ route: RouteInfo }> = ({
  route
}: {
  route: RouteInfo;
}) => {
  const [state, setState] = useState<TransactionFeeInfo>();
  useEffect(() => {
    setState(undefined);
    route.getDepositAndFee().then(setState);
  }, [route]);
  return (
    <div>
      {state && (
        <div>
          <br />
          Deposit For serum: {/* In lamports */}
          {state.openOrdersDeposits.reduce((total, i) => total + i, 0) /
            10 ** 9}{" "}
          SOL
          <br />
          Deposit For ATA: {/* In lamports */}
          {state.ataDeposit / 10 ** 9} SOL
          <br />
          Fee: {/* In lamports */}
          {state.signatureFee / 10 ** 9} SOL
          <br />
        </div>
      )}
    </div>
  );
};

export default FeeInfo;
```

#### `JupiterForm.tsx`

Add the [following code](https://raw.githubusercontent.com/DappioWonderland/swap-ui-example/master/views/jupiter/JupiterForm.tsx) in `./views/jupiter/JupiterForm.tsx`:

```typescript=
import React, { FunctionComponent, useEffect, useMemo, useState } from "react";
import { PublicKey } from "@solana/web3.js";
import { TokenListProvider, TokenInfo } from "@solana/spl-token-registry";
import { useConnection, useWallet } from "@solana/wallet-adapter-react";
import { useJupiter } from "@jup-ag/react-hook";
import { ENV as ENVChainId } from "@solana/spl-token-registry";
import FeeInfo from "./FeeInfo";
import { getSPLTokenData, ISplToken } from "../../utils/web3";
import SplTokenList from "../commons/SplTokenList";
import style from "../../styles/jupiter.module.sass";

const CHAIN_ID = ENVChainId.MainnetBeta;
interface IJupiterFormProps {}
interface IToken {
  mint: string;
  symbol: string;
}
type UseJupiterProps = Parameters<typeof useJupiter>[0];

const JupiterForm: FunctionComponent<IJupiterFormProps> = props => {
  const wallet = useWallet();
  const { connection } = useConnection();
  const [tokenMap, setTokenMap] = useState<Map<string, TokenInfo>>(new Map());

  const [formValue, setFormValue] = useState<UseJupiterProps>({
    amount: 1,
    inputMint: undefined,
    outputMint: undefined,
    slippage: 1 // 1%
  });

  const [inputTokenInfo, outputTokenInfo] = useMemo(() => {
    return [
      tokenMap.get(formValue.inputMint?.toBase58() || ""),
      tokenMap.get(formValue.outputMint?.toBase58() || "")
    ];
  }, [formValue.inputMint?.toBase58(), formValue.outputMint?.toBase58()]);
  const [splTokenData, setSplTokenData] = useState<ISplToken[]>([]);

  useEffect(() => {
    new TokenListProvider().resolve().then(tokens => {
      const tokenList = tokens.filterByChainId(CHAIN_ID).getList();
      setTokenMap(
        tokenList.reduce((map, item) => {
          map.set(item.address, item);
          return map;
        }, new Map())
      );
    });
  }, [setTokenMap]);

  const amountInDecimal = useMemo(() => {
    return formValue.amount * 10 ** (inputTokenInfo?.decimals || 1);
  }, [inputTokenInfo, formValue.amount]);

  const { routeMap, allTokenMints, routes, loading, exchange, error, refresh } =
    useJupiter({
      ...formValue,
      amount: amountInDecimal
    });

  const validOutputMints = useMemo(() => {
    return routeMap.get(formValue.inputMint?.toBase58() || "") || allTokenMints;
  }, [routeMap, formValue.inputMint?.toBase58()]);

  // ensure outputMint can be swapable to inputMint
  useEffect(() => {
    if (formValue.inputMint) {
      const possibleOutputs = routeMap.get(formValue.inputMint.toBase58());

      if (
        possibleOutputs &&
        !possibleOutputs?.includes(formValue.outputMint?.toBase58() || "")
      ) {
        setFormValue(val => ({
          ...val,
          outputMint: new PublicKey(possibleOutputs[0])
        }));
      }
    }
  }, [formValue.inputMint?.toBase58(), formValue.outputMint?.toBase58()]);

  const getSymbolByMint = (mintList: string[]) => {
    return mintList.map(t => {
      let tokenInfo: IToken = {
        mint: "",
        symbol: ""
      };
      tokenInfo["mint"] = t;
      tokenInfo["symbol"] = tokenMap.get(t)?.name || "unknown";
      return tokenInfo;
    });
  };

  const specificTokenOnly = (tokenList: IToken[]): (IToken | undefined)[] => {
    return tokenList.map((t: IToken) => {
      if (
        t.mint === "Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB" ||
        t.mint === "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v" ||
        t.mint === "So11111111111111111111111111111111111111112" ||
        t.mint === "4k3Dyjzvzp8eMZWUXbBCjEvwSkkk59S5iCNLY3QrkX6R" ||
        t.mint === "SRMuApVNdxXokk5GT7XD5cUUgXMBCoAz2LHeuAoKWRt"
      ) {
        return t;
      }
    });
  };

  let inputList: IToken[] = specificTokenOnly(
    getSymbolByMint(allTokenMints).sort((a: any, b: any) =>
      a.symbol < b.symbol ? -1 : a.symbol > b.symbol ? 1 : 0
    )
  ).filter(t => t !== undefined) as IToken[];

  let outputList = specificTokenOnly(
    getSymbolByMint(validOutputMints).sort((a: any, b: any) =>
      a.symbol < b.symbol ? -1 : a.symbol > b.symbol ? 1 : 0
    )
  ).filter(t => t !== undefined) as IToken[];

  useEffect(() => {
    if (!wallet.connected) {
      return;
    }
    getSPLTokenData(wallet, connection).then((tokenList: ISplToken[]) => {
      if (tokenList) {
        setSplTokenData(() => tokenList.filter((t: any) => t !== undefined));
      }
    });
    return () => {};
  }, [wallet.connected]);

  return (
    <div style={{ display: "flex" }}>
      <div>
        <SplTokenList splTokenData={splTokenData} />
      </div>
      <div className={style.jupiterFormModal}>
        <div className={style.title}>Jupiter</div>
        <div className={style.selectBlock}>
          <label htmlFor="inputMint">Input token</label>
          <select
            className={style.select}
            id="inputMint"
            name="inputMint"
            value={formValue.inputMint?.toBase58()}
            onChange={e => {
              const pbKey = new PublicKey(e.currentTarget.value);
              if (pbKey) {
                setFormValue(val => ({
                  ...val,
                  inputMint: pbKey
                }));
              }
            }}
          >
            {inputList.map((t: IToken) => {
              return (
                <option key={t.mint} value={t.mint}>
                  {t.symbol}
                </option>
              );
            })}
          </select>
        </div>

        <div className={style.selectBlock}>
          <label htmlFor="outputMint">Output token</label>
          <select
            className={style.select}
            id="outputMint"
            name="outputMint"
            value={formValue.outputMint?.toBase58()}
            onChange={e => {
              const pbKey = new PublicKey(e.currentTarget.value);
              if (pbKey) {
                setFormValue(val => ({
                  ...val,
                  outputMint: pbKey
                }));
              }
            }}
          >
            {outputList.map((t: IToken) => {
              return (
                <option key={t.mint} value={t.mint}>
                  {t.symbol}
                </option>
              );
            })}
          </select>
        </div>

        <div>
          <label htmlFor="amount">
            Input Amount ({inputTokenInfo?.symbol})
          </label>
          <div>
            <input
              className={style.input}
              name="amount"
              id="amount"
              value={formValue.amount}
              type="text"
              pattern="[0-9]*"
              onInput={(e: any) => {
                let newValue = Number(e.target?.value || 0);
                newValue = Number.isNaN(newValue) ? 0 : newValue;
                setFormValue(val => ({
                  ...val,
                  amount: Math.max(newValue, 0)
                }));
              }}
            />
          </div>
        </div>
        <button
          className={style.operateBtn}
          type="button"
          onClick={refresh}
          disabled={loading}
        >
          {loading ? "Loading" : "Refresh rate"}
        </button>

        <div>Total routes: {routes?.length}</div>

        {routes?.[0] &&
          (() => {
            const route = routes[0];
            return (
              <div>
                <div>
                  Best route info :{" "}
                  {route.marketInfos.map(info => info.marketMeta.amm.label)}
                </div>
                <div>
                  Output:{" "}
                  {route.outAmount / 10 ** (outputTokenInfo?.decimals || 1)}{" "}
                  {outputTokenInfo?.symbol}
                </div>
                <FeeInfo route={route} />
              </div>
            );
          })()}

        {error && <div>Error in Jupiter, try changing your input</div>}

        <button
          className={`${style.operateBtn} ${style.swapBtn}`}
          type="button"
          disabled={loading}
          onClick={async () => {
            if (
              !loading &&
              routes?.[0] &&
              wallet.signAllTransactions &&
              wallet.signTransaction &&
              wallet.sendTransaction &&
              wallet.publicKey
            ) {
              await exchange({
                wallet: {
                  sendTransaction: wallet.sendTransaction,
                  publicKey: wallet.publicKey,
                  signAllTransactions: wallet.signAllTransactions,
                  signTransaction: wallet.signTransaction
                },
                route: routes[0],
                confirmationWaiterFactory: async txid => {
                  await connection.confirmTransaction(txid);
                  getSPLTokenData(wallet, connection).then(
                    (tokenList: ISplToken[]) => {
                      if (tokenList) {
                        setSplTokenData(() =>
                          tokenList.filter((t: ISplToken) => t !== undefined)
                        );
                      }
                    }
                  );
                  return await connection.getTransaction(txid, {
                    commitment: "confirmed"
                  });
                }
              });
            }
          }}
        >
          Swap Best Route
        </button>
      </div>
    </div>
  );
};

export default JupiterForm;
```
### Update Style

Add the following code in`./styles/jupiter.module.sass`
```sass=
.jupiterFormModal
  position: absolute
  top: 50%
  left: 50%
  transform: translate(-50%, -50%)
  padding: 4rem 8rem
  border-radius: 1rem
  background-color: rgba(0,0,0,.3)
  .title
    font-size: 2.5rem
    margin-bottom: 3rem
  .selectBlock
    display: flex
    justify-content: space-between
    align-items: center
    margin-bottom: 2rem
  .select
    border: none
    padding: .5rem 2rem
    outline: none
    border-radius: 1rem
    margin-left: 1rem
  .input
    padding: .2rem 1rem
    margin: 1rem 0 1rem 0
    outline: none
  .operateBtn
    padding: .8rem 1.2rem
    margin: 2rem 0
    border: none
    border-radius: 1rem
  .swapBtn
    background-color: #fbae21
    font-weight: 600
    padding: 1.2rem 2rem
```

### Update Config

Replace `next.config.js` with following code:

```typescript=
/** @type {import('next').NextConfig} */
const withPlugins = require("next-compose-plugins");

/** eslint-disable @typescript-eslint/no-var-requires */
const withTM = require("next-transpile-modules")([
  "@solana/wallet-adapter-base",
  // Uncomment wallets you want to use
  // "@solana/wallet-adapter-bitpie",
  // "@solana/wallet-adapter-coin98",
  // "@solana/wallet-adapter-ledger",
  // "@solana/wallet-adapter-mathwallet",
  "@solana/wallet-adapter-phantom",
  "@solana/wallet-adapter-react",
  "@solana/wallet-adapter-solflare",
  "@solana/wallet-adapter-sollet",
  // "@solana/wallet-adapter-solong",
  // "@solana/wallet-adapter-torus",
  "@solana/wallet-adapter-wallets",
  // "@project-serum/sol-wallet-adapter",
  // "@solana/wallet-adapter-ant-design",
]);

const plugins = [
  [
    withTM,
    {
      webpack5: true,
      reactStrictMode: true,
    },
  ],
];

const nextConfig = {
  swcMinify: false,
  webpack: (config, {
    isServer
  }) => {
    if (!isServer) {
      config.resolve.fallback.fs = false;
    }
    return config;
  },
};

module.exports = withPlugins(plugins, nextConfig);
```
### Update `Jupiter` Page

Add the following code in `./pages/jupiter.tsx`:

```typescript=
import { FunctionComponent } from "react";
import Jupiter from "../views/jupiter/JupiterProvider";
import JupiterForm from "../views/jupiter/JupiterForm";

const JupiterPage: FunctionComponent = () => {
  return (
    <>
      <Jupiter>
        <JupiterForm />
      </Jupiter>
    </>
  );
};

export default JupiterPage;
```

Restart the dev server:

```
$ yarn dev
```

## References

- https://github.com/DappioWonderland/swap-ui-example
- https://solana-labs.github.io/solana-web3.js
- https://solana-labs.github.io/wallet-adapter
- https://docs.jup.ag
- https://github.com/raydium-io/raydium-ui
- https://github.com/yihau/full-stack-solana-development
- https://github.com/yihau/solana-web3-demo
- https://github.com/thuglabs/create-dapp-solana-nextjs
- https://www.udemy.com/course/typescript-the-complete-developers-guide

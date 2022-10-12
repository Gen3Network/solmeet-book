---
tags: notes
---

# A Starter Kit for New Solana Developer

**Author:** [@ironaddicteddog](https://twitter.com/ironaddicteddog)

***[Updated at 2022.3.31]***

## TL; DR

- What makes Solana so fast and efficient?
- What should I learn at minimum to understand how Solana program works?
- How can I setup my the environment quickly?
- What is the best practice to develop Solana programs?

## Before We Start

### Prerequisites

- **[Rust](https://github.com/rust-lang/rust)**
- **[Typescript](https://github.com/microsoft/TypeScript)**
- [Solidity](https://github.com/ethereum/solidity) (nice to have)

### Why Solana?

- It's a new blockchain paradigm evolved from Ethereum (more or less)
- It's blazing fast and efficient, meaning it will be cheap and affordable for average users
- It's a good investment on modern web technology (Rust / Typescript, etc)

### Why Rust?

- Rust is fast
- Rust is safer (compared to C++)
- **Rust can be compiled to WASM**
- (Almost) every programmable blockchain has a rust implementation
    - [OpenEthereum (Eth 1.0 Client, formerly Parity)](https://github.com/openethereum/openethereum)
    - [Lighthouse (Eth 2.0 Client)](https://github.com/sigp/lighthouse)
    - [Near protocol](https://github.com/near)
    - ...and of course, **Solana**

### Compare Solana to Ethereum, what are the pros and cons?

#### Pros

- **Parallelism is the secret sauce** making it blazing fast
- Efficient network synchronization using [PoH Clock](https://docs.solana.com/cluster/synchronization)
    - [VDF (in general definition)](https://github.com/solana-labs/solana/issues/388)
    - Verification can be parallelized
- Efficient PoS Consensus
    - [Tower BFT](https://docs.solana.com/implemented-proposals/tower-bft), a specialized PBFT designed for PoH

#### Cons

- Rather high [hardware requirements](https://docs.solana.com/running-validator/validator-reqs#hardware-recommendations) to run a validator
    - 12 cores CPU
    - N cores GPU
    - 128GB RAM

### Structure of this Kit

1. Hello world (for environment setup)
2. Escrow program using vanilla Rust (for learning core concepts)
3. Escrow program using Anchor (for learning the best practice)

## 1. Hello World!

### Goal

- Setup the development environment
- Get familiar with tools such as:
    - `cargo`
        - `cargo build-bpf`
    - `rustup`
    - `solana-cli`
        - `solana deploy`
    - `solana-test-validator`
    - `solana-web3.js`

### Install Rust and Solana Cli

- See [this doc](https://github.com/solana-labs/solana#1-install-rustc-cargo-and-rustfmt) for more details

#### Install `rustup`

```bash
$ curl https://sh.rustup.rs -sSf | sh
...

$ rustup component add rustfmt
...

$ rustup update
...

$ rustup install 1.59.0
...
```

#### Install `solana-cli`

- See [this doc](https://docs.solana.com/cli/install-solana-cli-tools) for more details

```bash
$ sh -c "$(curl -sSfL https://release.solana.com/v1.10.5/install)"
...

$ solana --version
solana-cli 1.10.5 (src:devbuild; feat:2037512494)
```

> Note: You may have to build from source if you are using Mac M1 machine. See [this doc](https://github.com/solana-labs/solana#1-install-rustc-cargo-and-rustfmt) for more installation details.

#### Generate Keypair

```
$ solana-keygen new
...
```

Or, you can recover your key from your existing key phrase:

```
$ solana-keygen recover 'prompt:?key=0/0' -o ~/.config/solana/solmeet-keypair-1.json
...
```

Config the keypath:

```
$ solana config set --keypair ~/.config/solana/solmeet-keypair-1.json
```

#### Config to Local Cluster

```bash
$ solana config set --url localhost
...
```

#### Install `rust-analyzer`, `Better TOML` and `crates` (Optional)

- See [this repo](https://github.com/rust-analyzer/rust-analyzer) for more details

`rust-analyzer` can be very handy if you are using Visual Studio Code. For example, the analyzer can help download the missing dependencies for you automatically.

#### Install Additional Dependencies (Optional)

If you are using Linux, you may need to install these tools as well:

```
$ sudo apt-get update
...

$ sudo apt-get install libssl-dev libudev-dev pkg-config zlib1g-dev llvm clang make
...
```

### Build and Deploy

- See [this repo](https://github.com/solana-labs/example-helloworld) for full code base

First, let's clone the repo:

```bash
$ git clone https://github.com/solana-labs/example-helloworld.git
...

$ cd example-helloworld

$ npm install
...

$ npm install -g ts-node
...
```

Run `solana-test-validator` in another terminal session:

```bash
$ solana-test-validator
Ledger location: test-ledger
Log: test-ledger/validator.log
⠤ Initializing...
Identity: D4kA7VzHnmVa9HqfL1gQzTgHBGYdcsaADFxZnJfLxnxz
Genesis Hash: AvyN2Hka7q3aBUFcNbEKERQxEPiKs1B3kVVUiXnbdCk
Version: 1.8.0
Shred Version: 59947
Gossip Address: 127.0.0.1:1024
TPU Address: 127.0.0.1:1027
JSON RPC URL: http://127.0.0.1:8899
...
```

Next, let's compile the hello world program:

```bash
$ cd src/program-rust
$ cargo build-bpf
...
```

Deploy the program after compilation, :

```bash
$ solana program deploy target/deploy/helloworld.so
...
```

> If you encounter an insuffficient fund error, you may have to request for an aidrop:
> 
> ```bash
> $ solana airdrop 1
>```

### Say Hello World

First, we need to modify the `PROGRAM_PATH` in `src/client/hello_world.ts`:

```typescript
// In src/client/hello_world.ts
// Modify PROGRAM_PATH at Line 43

...
// const PROGRAM_PATH = path.resolve(__dirname, '../../dist/program');
const PROGRAM_PATH = path.resolve(__dirname, '../program-rust/target/deploy');
...
```

Finally, let's make the program say Hello by sending a transaction:

```
$ npm install -g ts-node
...

$ ts-node ../client/main.ts
Let's say hello to a Solana account...
Connection to cluster established: http://localhost:8899 { 'feature-set': 2037512494, 'solana-core': '1.8.0' }
Using account 4h8EgjxFHnTLshhGWb91MgyN2PXJZ8dmbc8UiTsfatLf containing 499999999.14836293 SOL to pay for fees
Using program 7hV1hUKgY4ZF3J2UYAhvZFhNtr4PB4MLufsuXFU5Usa2
Saying hello to f5nadW1a9e86aaigWfuKPhAKTYCNYUeZ9xm9i3HjS8P
f5nadW1a9e86aaigWfuKPhAKTYCNYUeZ9xm9i3HjS8P has been greeted 2 time(s)
Success
```

If we take a closer look to function `sayHello`, we can see how a solana transaction is constructed and sent:

```typescript
// In hello_world.ts
...
export async function sayHello(): Promise<void> {
  console.log('Saying hello to', greetedPubkey.toBase58());
  const instruction = new TransactionInstruction({
    keys: [{pubkey: greetedPubkey, isSigner: false, isWritable: true}],
    programId,
    data: Buffer.alloc(0), // All instructions are hellos
  });
  await sendAndConfirmTransaction(
    connection,
    new Transaction().add(instruction),
    [payer],
  );
}
...
```

## 2. **Escrow Program (using vanilla Rust)**

### Goal

- Learn Solana account model and core concepts such as:
    - Account model
    - Program Architecture
    - Program Derived Address (PDA)
    - Cross-Program Invocation (CPI)
        - `invoke`
        - `invoke_signed`
- This section is extracted from this awesome tutorial: [Programming on Solana - An Introduction (by paulx)](https://paulx.dev/blog/2021/01/14/programming-on-solana-an-introduction). Some of the explanations in this doc are more comprehensive and clearer in the original post. I strongly recommend you to read through the post at least once.
- **Program Architecture**
    - `lib.rs`: registering modules
    - `entrypoint.rs`: entrypoint to the program
    - `instruction.rs`: program API, (de)serializing instruction data
    - `processor.rs`: program logic
    - `state.rs`: program objects, (de)serializing state
    - `error.rs`: program specific errors
- See [this repo](https://github.com/paul-schaaf/solana-escrow) for full code base 

### Core Concepts

#### Account

![](https://i.imgur.com/7kUb9di.png)

- Accounts are used to store state
- Accounts are owned by programs
- Only the account owner may debit an account and adjust its data
- All accounts to be written to or read must be passed into the entrypoint
- All internal Solana internal account information are saved into fields on the account (opens new window)but never into the data field which is solely meant for user space information
- Developers should use the data field to save data inside accounts

#### Program

- Solana programs are **stateless**
- Each program is processed by its **BPF Loader** and has an entrypoint whose structure depends on which BPF Loader is used
- In theory, programs have full autonomy over the accounts they own. It is up to the program's creator to limit this autonomy and up to the users of the program to verify the program's creator has really done so
- The flow of a program using this structure looks like this:
    - Someone calls the **entrypoint**
    - The **entrypoint** forwards the arguments to the **processor**
    - The **processor** asks **instruction** module to decode the instruction_data argument from the entrypoint function.
    - Using the decoded data, the processor will now decide which processing function to use to process the request.
    - The processor may use **state** module to encode state into or decode the state of an account which has been passed into the entrypoint.
- When writing Solana programs, be mindful of the fact that any accounts may be passed into the entrypoint, including different ones than those defined in the API inside `instruction.rs`. It's the program's responsibility to check that received accounts == expected accounts


#### Instruction

- If you are familiar of Ethereum, think of Solana instructions as Ethereum transcations, while Solana transaction, which can wrap multiple instructions, is anologous to Ethereum [`multicall`](https://etherscan.io/address/0x5ba1e12693dc8f9c48aad8770482f4739beed696#code)

#### SPL `token` Program

- The token program owns token accounts which inside their data field hold relevant information
- the token program also owns token mint accounts with relevant data
- each token account holds a reference to their token mint account, thereby stating which token mint they belong to
- the token program allows the (user space) owner of a token account to transfer its ownership to another address
- All internal Solana internal account information are saved into fields on the account but never into the data field which is solely meant for user space information

#### PDA

- Program Derived Addresses do not lie on the ed25519 curve and therefore **have no private key associated with them.**

#### Cross-Program Invocation

- When including a signed account in a program call, in all CPIs including that account made by that program inside the current instruction, the account will also be signed, i.e. the signature is extended to the CPIs.
- when a program calls `invoke_signed`, the runtime uses the given seeds and the program id of the calling program to recreate the PDA and if it matches one of the given accounts inside invoke_signed's arguments, that account's signed property will be set to true

> To spend Solana SPL, you don't need to approve. Why?

#### Rent

- Rent is deducted from an account's balance according to their space requirements (i.e. the space an account and its fields take up in memory) regularly. **An account can, however, be made rent-exempt** if its balance is higher than some threshold that depends on the space it's consuming
- If an account has no balance left, it will be purged from memory by the runtime after the transaction (you can see this when going navigating to an account that has been closed in the explorer)
- "closing" instructions must set the data field properly, even if the intent is to have the account be purged from memory after the transaction
- In any call to a program that is of the "close" kind, i.e. where you set an account's lamports to zero so it's removed from memory after the transaction, make sure to either clear the data field or leave the data in a state that would be OK to be recovered by a subsequent transaction.
- Solana has sysvars that are parameters of the Solana cluster you are on. These sysvars can be accessed through accounts and store parameters such as what the current fee or rent is. As of solana-program version 1.6.5, sysvars can also be accessed without being passed into the entrypoint as an account.


### Escrow Program Overview

#### Flow

![](data:image/gif;base64,R0lGODlhWwHxAPcMABYWFhwcHCIiIiYmJjIyMjU1NT09PYmJiaGhoeXl5ezs7P///wAAAAgICA0NDRISEioqKi4uLkJCQlpaWmZmZmxsbHd3d3h4eKSkpLa2tsXFxfb29ktLS319faurq7Ozs7u7u+Li4unp6QcHBzs7O0VFRU1NTVFRUVZWVlxcXGJiYmlpaXNzc4ODg4SEhI6OjpCQkJaWlpmZmZ6enq6urr+/v8PDw8jIyM7OztPT09bW1tjY2Nzc3PLy8vv7+zg4OEdHR09PT1VVVV1dXWpqapSUlL29vcHBwcnJyc3Nzfj4+AUFBUlJSVNTU2BgYHJycn9/f4GBgYaGho+Pj5KSkpubm5ycnKioqKysrLGxsbm5udDQ0NTU1Nra2t/f3+Pj4+jo6PPz8wEBAQQEBAoKCg8PDxAQEBsbGysrKy0tLUBAQFBQUFhYWGNjY2VlZW5ubnBwcHV1dXl5eYCAgIWFhYyMjJOTk5iYmJ2dnaenp6+vr7S0tMfHx9LS0tfX19vb297e3vHx8fT09AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACH/C05FVFNDQVBFMi4wAwEAAAAh+QQEyAD/ACwAAAAAQQHxAAAG/8CFcEgsGo/IpHLJbDqf0Kh0Sq1ar9isdsvter/gsHhMLpvP6LR6zW673/C4fE6v2+/4vB4d8u3/gIGCTDEQDAwmOgs8HlUcG4ORkpNaNgw1PjgcBQsfAlMIJQwKlKWmp0oIo0I6LTwCiIsGACmkBy0ntD1EISiiu6jBwpQ7DAMtmEIuADgiDBU2JgYLKwwwIMdGOavD3d5/rQGHCJ2fLwGsoyslQh7TRdvA3/P0dTooo57Uh/w5KyzptHGrR7AgmhQqhvC4pK8Dp102fFBIuEAGAIGkDGrcCMZishb5LoK4tAGkRAd9DNSCN5Cjy5dWKvAj8GERgAH7jIGgNuAQgf8EGGEKHSqlGRIROSBRu7ChD9GnUMX8i0q1KpcMyaxq3cq1q9evYMOKHUu2rNmzaNOqXcu2rdu3cOPKnUu3rt27ePPq3cu3r9+/gAMLHky4sOHDiBMrXsy4sePHkCNLnky5suXLmDNr3sy5s+fPoEOLHk0aMr/TqFOrXs26tevXsGPLPl06S0u0t2tLyW2Wt+4nvskG/81kuFjjxJMgB7s8uZHmXqE7HyKda/Xp17VmT769avff36OGrz3+aXnS54emF70eZnvQ713G9zx/Y33O9w3m17yfYH/M/9EToGUDflMgZQd2k6BkCwrToGltPfiYhKdQ2JiFpWC4mIaTcBj/2AvyUJdKao9AMYNSScRDxIkLkEDOENgkEcMJVtCww27TKTHCCBaEOFwCCTgEZEbAAaWEigG1iAE8MtJYRQSNROEhYDA0sGOPQiB3zhArwCABAipkIEQKOx0wQAAttGiMCBIJQFM7BBRgwUCzRCACCevocgONM3SgAgU8lYDQirjoMuZNMghRwAwQyIQmCX74QEJEOuao42lMaQnBEBLE8qaSNzCDgQMajERDmx6oYsNCFagykKmSWhMDQxctU0INHjjwwgGxcGlNDQLAwJMNqvQxTqiukFSDcVN+1els0J6WSxNbKkrRpzTliklSCqxizZit2llRS6vgqagMMS4D/+6YTq5bTic34kDSt0ouRU27yhEIxgOY9qApp4nWW28Kh1SwyygJoOaLwe8qVO6LX6Y7QZZRHoDvRAEBa8y8itQ7qg8BRFlpZe+dcyUw/56r8gKX9KGAtt0CRVJTO+DSDrlAuXiuxCtjzCWg76JJCqk+cAMlxcg1G90XVjJFRMoC+/LyJV6SYoCw5aKggA5CE/3L04roDGqtE1f00zYXo5SAuQ40MgPHPfe6hNJdlRxiltRuuvJIiAzgQQLiOACpmkAG7mQuiLREwihiR0w2K4YgcnFPa44rOAcocXOBsjPPra8Z4bHJi5HaOMWK6FfcaMRUpp++hD5N0G3dGbLH5P90FGYKG/vnZdReBVagiOmE71b9R3wex1NlfIS8k5H8Hc9DtTxb0ZtHO/MkX09982NUT4f36mm/FvhCTT8+92KQH4f6L5mvFvvyif8++mHA74b99smfFv4aub8//fvCHoL0hxsAMk2Ak/FfAbMHOgQyiIBn4Z9+INgbA3ohWhjMoAY3qEELWoouEvygHkIoQjyQsIR2OCEKv7fCvKiwheuD4V1eKMP71fCGOMyhZ2iAr9QJ71BKycYQWKTDYfBwCzPQ2tM6wBMUsYxbRaRDBkgQLEgwqooKwNNNbkSDCERAagrB3K06QUUQnYRMsqCFDoqhKyIsyyNEmJM5zsTEKL7/gVRr+0AISAUsBCSsA2sDyK8ggK8SuKAkNMIjBz4Qig24Yyk8sAA6ZuTEfaSpCMwoGgZ2wIED2NENm4CBIrwkAxlwICGrSCLsXtBDGliME6FURKigAYkGtCAGs2KEEomQC6CBDVhJ+iQbMKCCS1jARbgUUypPkEQ48dKQZmpHMcWEgxYUgBOywuXLdnkzS/wQb81smDDV8AlI6i0FMUiYM7MGtiaWswIWuKVNsngiDZjDBLwgyTlQ90Rn8EAidRxnGuQorgqIA1LOcKajJDA5AVCgbQTdwQ4CQMc0fmsbAS3ASiTFzQmghAU++adA1TCk0aWISCZVSkldp5C7FUWk/yONqUxnStOa2pQSRHQCCKCIBAgYZRE1OSKMSmQEoUohpzfdQiafgM4l+HSIWntkSotgVCnxNKlYkOMrrJmBFShDTA+dwA1yxYy1qfFQHv3pTVzQxV460mAosKYIuvjFdvHglIa6h+CSEleHGqMDdQQsVq1QVkTgQKrMxIAEHAmpuAK0ZpOUQA6W8VNK0uBbAmAkO+K0k0Ha9Vt/Ghg+OauAkFnCD0QbLBUKCxTFjskDs6yJvXSFS8bJtnON7OJrj/imVX7WDzkopysNAMsoZdK05VTtFFhLRpVVM06zZas2l5ra5gpVBdfYbETCaUqHBURQfSou3pZCgUMqd7nNWP9FjBaCARd4Up3YrZcNPgHPTuAWn9fFpXZtciNBOewG9/LXjQwqsExyrbrnhUJajWYMCPyNolX0iD3FsaTgNsBNUMRofnlLqYV+9kwDUISjHuqODqOkRYtNsBZIx9KptngKrQtmS50qMhUPg2tR6AVObPwNSpkoUjweYZCVN2TxFFl6R7ZeksO35PI12clPbl+UpTxljtCwyk/Dcv60PEEuF+TKXAazlsWMZTJX2cxTRnOU1fxkNjfZzUuGc5LlfGQ6F9nOQ8ZzkPXMYz7b2M8qBnSCBY0gDhr60Ije4DwSzehGO3o1DnRQhAgNBRJSGjiXHt6iJx3pYGQ6dp8uzqb/qRdqzxmI09s7Nak7jYpSX4rVFUL1+VQ9Plfni9bvszUSLC3r+eF6f7o+Aq9XnWpvBPs5x8bkqGsNa1Mk+2nPFtGvcRPt8Rq71/+bdgSrzbJl57rZGcL2Aq9N7FmTm9nFVpC41xCBE7/LZnsY9h1ncsmiDu57EtrGzE7Lg44JWdvkrHAuj+DaOhwCDhZYWED6lEaxLgIfY7yjt9uQ2oTtRK8F0EA5CDYpORz8joxUrwPMtgE1miAh16zB5lC6Bnnfr0te2nEAFrnyUYmVikC+34PWe9GR+ykDq4ikvEQKgFPZEOBpuJwoAqADoKMMA7mCiIU+3gaOw4Idp/VTIVaE85oYSBzpaKjuKbd+roL3Ew5UZ8OvVB6SAGtuJEb6KcUnrnbZ3uu0WSrVjotRSbU3SBVA1lXWa9HGdXQLRED3t9rp3nJRhgNEbXT6qE5VAXasr0H+5dKkfJ4Qi1RuVAVDO+PJSW8/zMoB3yKrTjxOPBbTLIZgj8NVF5HzeYObEtxOe7bP/e10DyP3uR99WXQ/bnWX29e8B/btO7TuCsb+OMF/fliIH0HhCyf6yaf28iUB/O1Hovu+l/Txd298dJu7/L0//++bP3zrj4X6zs/+tr0/CPCrX/zmRz76lR9+T7NfOO4HfY82gARoaItWgAiYgLIRBAAh+QQFyAAcACwQAAAARQHxAAAH/4ALgoOEhYaHiImKi4yNjo+QkZKTlJWWl5iZmpucnZ6foKGio6SlpqeoqaqrrK2ur7CxsrO0tba3uLm6rF9Ku7/AwcKQRRAMDEFdC19Xk1dCQlgbw9TV1p0aDEZKW0wFC1kCklkMTxgCF9fq6+yNVgwKglxRXgLI4AYAQ/EHUUH6YQg9ISKICoB2CBO268JgQJRtBQEkAVPuiLcFRBgUMYKOEJdeGIUoHEly2LwAxxCAEzclgDx4RIAUlHCIgraSOHPq4jIBXjiMx4JuIfLkZaEkAAhw0cm0qasJTgZ90fYTyrdpGpQQjejRwQFfTsOKLVXlplafB7XcjHIwY68f+/8GUTDQpa6ysXjzbqoQlECWZUmBNtw295hSQgOCHtPLuHGliYjALBW0FaTjy5hLbc3MufOnDBA9ix5NurTp06hTq17NurXr17Bjy55Nu7bt27hz697Nu7fv38CDCx9OvLjx48iTK1/OvLnz59CjS59Ovbr169iza3+kuLv37+DDix9Pvrz589+3v4JHm716Vu5lx3+faj5s+/RN4Xe9P/+o/qwB6B8oAqpW4ICdHIiagghqwqBpDzZ4SYSkUSghJRaKluGFkWzYmYccOgJiZiOGuEiJl6FoIiIqNtbiioW8qJeMMApCI143wpijWDua2KNTP3IYJFNDSlhkTkcimGT/SUv61+RIT9IXZUJTqldlO1dqlyVY1myJHYBXeMdEJFZMo8gW85W5wA8qDaJWIkU0YYk0kuD3VY2WCJhAAlbtGQ8kBXIxH3tsHgWnnJVE0EyHhViwRAN45ukOBHJRIQECQ2RAmRYLHDBAAFKs2RAYWgng10wFOEqIBKMaEBNASMhpBRROUNDpAEDUKhCtAFGWlJoFIABBRqD+ABZdJ8rVwKNURFrJgS0NwioFXZy65hVISHQOEmpJc4EAWCCgzVQVCBtft0rkU0URVB0kBTJGhDnFu4j6ulFHFfygAQbspZQtPTcZUSBMywbl7LOTShvVtYL4FeY2W5Dab5tDlEsA/1dSEQpFw1a8yRZlRa1Zb0gzgeMFNzdpxDHJAzGCzKPoxdyXkgmvzDDDThxTwTTwJNAdNDuvFGMCN1/q8QQ2LnpAELsaxVFDKU926jkLBLCoIgQX/ACBNDMSrc3WjksqFgBUocDEWHlBK8ZGFX0vyUWPTMHCZVUdtAPbuKdo0ge6dwHMzXri5WvQUmozNAqUdYWl8RjQLKFIc2F1ttsAMehSjnN8NFdoyu3AW/s00Mw7eUttNlBIu9zoo58Mzl/NN6v1cjMoOWCsqHvWjmgTx/xTSD4JFHrt5oD1Lndio3Zq/Od+B2xWsoZM0XrXlpDqkfWG1HV9Jnc1asEGlskjPv8iGRzUSJGut5b+XulI4mngqutHfUmgOfOXiKesv5r+wqA/f2z+GxD/ghFAJ9WmgPkZIDAQKKUD5u9/93mgAB0ovwm2R4IGvGAFMzgbBr5Hgb/woJUoWAoQQoiEpDBhaVSYCxFuh4W4cKGWUPgfCBIOgwmkoShgqCEdhoKHngFiLWSYHSHSgohfkpkSl8jEJi4RhwdrjhGjyCQqPmeKVlQIFrOIkC1ykR1e/KI6wijGLpUxOWQ84zDSqEYCttE4bHzjLuIoxxbWcTh0vKMt8qjHI/YROHz8YywCKcj1FLI3hDwkfBS5m0Qysj6PzI0jI7lBSmrQkpfEpHw0mUlOevL/k6C0DRZGdoktaEphcrnYINQUSmpkgZRzEombzPK8qkWslavIwA86cite/uBbBVgK2SKAOKkwwXYT0SUvbTIBJCyDd0D4CEpClco1fU9ZLPnUxnBZwsplYSoaOAK4KAKFLwShKBpBysiAECopyAlvwcuAsDaABWMRRW0uiZOZBhEA3w1NCV6p1p24KQpvUGEplipCFZgQFfZYoQk/6RTTCBEuA3wjCAQ46EqIcD+vrAseDzUEOU6JGC5wRB6GI6goMJAzLViATQrllEMhOiahKWxWV2FpwKRAgG9URKGJk+U/hxZSm6qUaxixgAYMN4QiUKRkkMvYUvKlAHEkNQp3/4IHE8q0VCUUtSt/KilFTja3o4bCUQWIgDL4YjuelYytEjgeOvCGVrVKTpv4AIDztilVQ/QEDE8wzMnMCgo/bS8yYT2sLww7PqkEpHqDJaxkJ0vZylqWoKx0hBZumQgIcNak+KCoTA4xSkpk9rJ14mwjnOBURXh2lSIprWJFS4kAqBa1j0BrPfzh0k2VqplhkkgCoCm1fkoEpR4lwD9kRRAhfGyYxXxmRoLWDWQmda7o2CYUpIdbl2UFXs+gzOJoEt4JSO+equLtAZ7HOLJhQHLylElPZYoAdWbsvOlgZxjcuSYIaEpbkAJoVrqbrLMRjaWCCAK2KjKNpgJ0u+z6Qv+wbASZ0JINZKWdWgRK1rYtWHW83whbxCZnVQK7VpocLlQSeHpRp/7UxQP+HGxDm9QMZ6WoC82YUXK1rhAfgcJXjYKJTzQR9mhBHF7QCFYBowDWMqyrFdiZWiocUtmy1sZAzlVfnzCmtfgYyIKS8ZAT8Ve9NcS/1cKr4paqVwwUz1Sf9WhNryzfAcN1oss48wCmmhSb1NPOMk7rmCVBtNk61tCRCF/bEP27qw26GpJLdBNS+mhr/JhMXKo0LCap6UFwutML+HSnRa1pUlfa1I9G9aBVPWZWD9nVJoY1gWXdXVrj1taoxfVldW1ZXlfW15QF9mSFLVliE9bYZuWjE5f/zexmp+dDzo62tKftnRN+yIecwTWy41ehIGKbRN7uZLiz/W3MaLvcKRr3JleobgBam9zivra8O/hucMcb3vhed7fnre/RnPve9s63u9nN74HvW+ARJDjCb6jwgPe7hwVP+MEdbnB/t1viFo84wydubnQ75t/0DkUExAzluOgC5K5wQF+ETFoD7DEUaDKLOJcxmZNffBXwBIPiWv5yUFgAaEaZVfGa+cykVLgVKEc6nSjCKZ5Y9xxDsO4gRZ6F8gVk5mvThwbOKTIj/C2xOL+5KhzA0TgNwBf9PAIUPhemZlo000gHxUk34Lwa+w0pgy2bITU+dmJazrZvSlqYrr6l/BtF3VT3wLp5zz5LxbQ27nyvz/3SZY6UwvR2Ap566w4aL7TYfbFUCDz4wD52sUs+lTOn8BWsypB9pvwzYgZoFRQv+w3cU2XkqPkiI4+KgG7hXY8z25v45a3RbvoTWka9A0hWluQFV2V7X/jYV+4Ldqmcq/YYjCyqVOhDH5/3sbjtMuD++oZ3HODnl7766p3+h5t+Rh530fv3x/50o9/+6g9Q/T8ef8YkHeMQl3/0Z374537ghyP9B38HmBrC9n8bl3ECaCD7J3/3x38LuCAT6H8JmBcO+DoEaIEhx3EFWHEBSHEAOH8IWIEUGIEMSG0u+ILhEUQwOIM0eB6BAAAh+QQFyAAMACwAAAAAWwHxAAAI/wAXCBxIsKDBgwgTKlzIsKHDhxAjSpxIsaLFixgzatzIsaPHjyBDihxJsqTJkyhTqlzJsqXLlzBjypxJs6bNmzhz6tzJM2cIQT2DCh1KlGMRNGLEBPGzAJCHiUWCtNFTtKrVqyyPiDGiJAmHAgs+CJB49ACerVjTql278awCgX/mABKgtKkBAEPe1ukQBG8ggkHqCAQSh63hw4gX/hEzYA5XgXQAJEkg5s2RrwuIiCkCQoAdg3/OgkhMujRiP3ACJEUQdqydM3DFJHADRGAeAwaHJO1jurfvtH6EyBabOanxPkTgxEYIBPfv59BzDmkzENBW4nLAAj2iJLltAIAf3/8JEL28+Zh30HYfDhsE2jmw3ZQBlMBA3oFtltbvcL6//5SaJUXAB00BMEBxu2U2gIAJEJQEAEkJ8dN/FFYY0mQIicCUQG7IIciEoP1l4YgkAlhYiSimmNVjKrbo4oswxijjjDTWaOONOOao44489ujjj0AGKeSQRBZp5JFIJqnkkkw26eSTUEYp5ZRUVmnllVhmqeWWXHbp5ZdghinmmGSWaeaZGhmn5ppstunmm3DGKeecdNapJppDidGjnngGxeeOf/a5U6A5EiooTobemOihNS1ao6OMygTpjJNG+lKlMWJqKUuavtjppil92qKooJpEaoqnljpSqiWyqipIro7/GOurHc1aoa20prlnrpLuyitMuP4X7K8VDdufscRKhKx5yyb7ULPRQessQ9I+V+20CV3rm7bYGsStad92O1C4pJErrrmIoYutuoax6yykHrDJQUR4AKVQH4TWu8APeTgIHkJ3rGGRHn8oK26ois0mQAIMRySbYoQ+bABrDiYUlUVpPOXwwShhWgca+NmhBgJD7MEhVwcMcMYc+zImQncCDGgbAQXEEehdaYhgnxp+ISEwHny5oSBhAg+0V1/3iSDfxIPhgYZmAaSsRNMLuZusxyAPVpfM+3rAh2TxIuEeVdnpgcBWIVR2dqBjK3HXHUVcB15kQBiRRxl71YXfZp19//bGD3yc3eBqSHi2GFdGVGp1kHfZ6fjjdzL02kBqUNd102F3NfjgFE/nNGQRN/gDf10nriDV+xbNYW2thUXfg1xtJlDG+76RmRDUQq677s7JuPhBWFN+B+po9KtbGbYrIFvaagrBRvJIhH556XOzIVAZGtehembKtW5EGoyh9fDl8S4QgMYK/Z4+jeoXFDzxFKPNsB4ADP+wen/EFcR37otO8cidWQAdrDc92hDEOwIETwCShz0ljI92ykuPvdank/YN6ibvK+DP+DavrgVCYhLyQ9RgpzPp8Qtz1YMMAfLnl73Rxz6CaKBoHLghAhRBawSs2gUphcGGfEx432HMAP88kADVlOEHb/nB8oxYtDVEiFBKrE+/qCfApCFFKduTD4MSeEQOlIGGoHtK5qi1Q9/18CIvq1iGQNSUBmGkYLnxEBsV4MY0HoQ4DbEg8Nh3RtJ0CCqNcYgevcVHmwyyLSyCSB1MJsgyZqqPOjokQSRpSEgWypEwOiQl/YRJT1kSR5tcQChnoklf5WSUvaokj0KJypiUcpWddNErARXLUX1SUbVU0SwjmUtU3dJGrCxko0yJKGHSpJWn7CWKdnnJChqTlMT8pS6l+UxVmtGavHQmD7HZzGRuc5iw1OY1wUlLcT6Sm6BUZquo+c1itpOUu4unPOdJzzWpk0TILFU+XbL/z031k1McA1Y1A4qwdxLUS/88KLMUKpE3fEYjIODNQAYgtAIRyGhVM04BNtS/PDE0InB4aEaGcMPqoOUHlltO+nhTnxxOEhAe5VjKmAYzmYFgOsj7C3zUMDKC6EGJuEsgTZ0YhwYVzi/xkswBo0YeghSgLjst6bjg+AbcBQd5TCFDG/CChJ4ktEf40hB5buMHwdHvA304A1rBJrtxjSYwYU2r0lgWBw8ZjgPUEYJgCvLURBaIjgHgw/e6Oq415ActUgkBHOZ1REDAx6sBxcv/2AC3ESKRe3LoHgdE2hQEwMeua6DYxxzTlc3c4Q3xkeq42mrSKkImedezwwcQ8APc/4xPNg0E4z1plYCoyCYNbbBDETjzUw7ZgaSou54b8vC8znrxZTflKt6EWy/kLlVw/uLea18aG61MLYZFhOP4PloTIxRtiAewHBqS4IEOBre4Isht6wRRs5tObYgcQNwK5+Na637nJ0FQQ3bxyLXrsVQOCqTK58hwgPmS1yZKRMO8FFaZsLj3M0o8ompptsD5RHhe8aIZgZxmIN6kh5FF1FeKnYqbDHdwtQLiQ1iOY76nqucjIW3LW7o70RN1liFp657ytKslOFbHjgcxckHmqORAcBQu360IfR7S5DqCJMe6IsjkEigi1y4kMl/86xwfrJOP0TQOAZBZepVIup321P9oKmvzAIbawqPm1WVOlYt8/3CGnOomvwYRwFn/usjWtBDNBSazS+I6VsAJDrV92AN7jiBpkX6NPvNh9FwDkdnS4RW8S8ZiQS6mvdD8q7UI1JNDaRiCjJEVu9NCpWRfJpXKkoxl5kOOZjnbh9OKL7QvG62MHwQ31OY6N+O1De6GShX8jAzBf1E1Hb4H5VqPh2Lv6klvnwtc6iJnr5KxLmUHkjjHFM+5D9tDauIgluEO99gFga+WcafU6RXIeVtljbTxwDrABne4Es32TuzbtfTeNwmrzjV85dtp60T0vB+oW+v4qz14z9sgJNNuWMl9IBV+ULsSA4HBdZaEdfXkw13/6UuFEx5Y+gpIqiJEA3BzhjMQH1HQhlYqG/jr05yFWs8QwnVywyrt4rRQ5bAVOE+qjOQQJXnJSH6y5i4idSk3/WqK3tJXs16ugXK9Vl7/+ka23jGxi4TsJ0E7ntReErafye2rMntI4H52ucMq7Ha3CN3nnneP7P3ufefI3z8y+DAVHuyBHzveE78xgzKeIofnSO8er/fFUz6Plr+8Dh2v+WdlvvMIibziQT8R0WeZ9I0fJ+o9z/nVZ+vzrr8ejQIw5tiHfva1tz0hZ7Qw3ZOR9032/e5lNICqC3+SuD/+64Gv/Nszv/nDj9EKoR99GE2f+sh/PvZlr/3tmz4jgNv+/7hopAbuiF+U5Pcr9b+PkfKfH/0zkvj52X8RQM8//e+n/0QYGYSLThD6+hcRi1E8/WczHbd+MaJFZEBj2BeAArgm/XZ8dcBg6PcaDdYiqMWAzTcGCxg5siQg2IdgauIhL5IaZjAaDeiBnpI1IciBY0CCMCJb85cUXfZ+MIJgMGiD0leDXlVPPviDQAgnrRKERFiEPlh96YRP0QR/5SQrS+iAgrdO4ZR9TWghUDh6VJhNTjiF48eFt/KESygsYOiFFHKFp9eFVfiFZGiGGcGGWdhNaKiFVjiGaViGdCiHaliHbngRe8h9ehiG/tGHxYKEuKSEawiIx3KHcJiHeMiEjf8ohodIhpD4h5IYiIqYhFtIiXU4iY0oiJAnhZr4iJYYiS8BPt/lXXzBE4L4Ox3IGEIXb5d1TISYEviCP180ZbtlLa7EGiIQNzKGcbEITe4DE3HwPATkXUBjF1jVFMIhf/wEivzUbJQxGle1Ua3xZwF3KbPYMbOVbl+UimcwFVKROkZgMzvWEqvoSkQANDxzX/nVAd+oNrUVZc84jIsWZrKDiiUzPnIQPTBlPugDUIZ4KWnQHCUmaSKCPeXTFMmGjttoEjsXM3Whj+PhU2uiWiuRjsAijXh1FMJzG27lSg/ZdrJlN+xRRabTWe6RRhhSj5kYjc7mXQZ2NyolUPbIEmfdE2UK+Y150UDJEV+C8QENqRIaeSl24AeogTcAWVpKcDdkE4EuGYcqQRhLpY9cZI1JxVoOOZD8pFEnEjdlkI90kSAieZM7YWVOdo5RyYg3kY1QJoxviIlzSIqimIh0uYh2eJdyyZZ76Yh4yYl46YmlB42BiYjnIZgGY5Z/OYqhuJh22Zh9CZh9iZipN5eQWYgvWZiV+JidaJgLpZeYaZmduZmHeYmhyZen6ZeRyZijuYmsqZmuyZmwWZelCZrARJiT6ZnlQZkQYSi8yXqZOZlGOJzE+ThDWJzImZxuEhAAOw==)

#### Account Relations

![](https://i.imgur.com/0r1svM7.png)

### Part 1

Fisrt, let's create a new project `solana-escrow`:

```bash
$ cargo new solana-escrow --lib
    Created library `solana-escrow` package

$ cd solana-escrow
```

Next, we update the `Cargo.toml` manifest to as follows:

```toml=
# Cargo.toml

[package]
name = "solana-escrow"
version = "0.1.0"
edition = "2018"
license = "WTFPL"
publish = false

[dependencies]
solana-program = "1.6.9"

[lib]
crate-type = ["cdylib", "lib"]
```

According to the program architecture, we will have five modules in the end. Let's create all these files at once before we start implementing them.

```
$ touch src/entrypoint.rs
$ touch src/processor.rs
$ touch src/instruction.rs
$ touch src/state.rs
$ touch src/error.rs
```

Next, define these modules in `lib.rs`:

```rust=
// lib.rs

pub mod entrypoint;
pub mod error;
pub mod instruction;
pub mod processor;
pub mod state;
```

Let's begin to implement these modules. First, we define instructions. Instructions are the APIs of program. Copy and paste the following snippet into your local `instuction.rs`:

```rust=
// instruction.rs (partially implemented)

use std::convert::TryInto;
use solana_program::program_error::ProgramError;
use crate::error::EscrowError::InvalidInstruction;

pub enum EscrowInstruction {

    /// Starts the trade by creating and populating an escrow account and transferring ownership of the given temp token account to the PDA
    ///
    ///
    /// Accounts expected:
    ///
    /// 0. `[signer]` The account of the person initializing the escrow
    /// 1. `[writable]` Temporary token account that should be created prior to this instruction and owned by the initializer
    /// 2. `[]` The initializer's token account for the token they will receive should the trade go through
    /// 3. `[writable]` The escrow account, it will hold all necessary info about the trade.
    /// 4. `[]` The rent sysvar
    /// 5. `[]` The token program
    InitEscrow {
        /// The amount party A expects to receive of token Y
        amount: u64
    }
}

impl EscrowInstruction {
    /// Unpacks a byte buffer into a [EscrowInstruction](enum.EscrowInstruction.html).
    pub fn unpack(input: &[u8]) -> Result<Self, ProgramError> {
        let (tag, rest) = input.split_first().ok_or(InvalidInstruction)?;

        Ok(match tag {
            0 => Self::InitEscrow {
                amount: Self::unpack_amount(rest)?,
            },
            _ => return Err(InvalidInstruction.into()),
        })
    }

    fn unpack_amount(input: &[u8]) -> Result<u64, ProgramError> {
        let amount = input
            .get(..8)
            .and_then(|slice| slice.try_into().ok())
            .map(u64::from_le_bytes)
            .ok_or(InvalidInstruction)?;
        Ok(amount)
    }
}
```

You may notice that there are a few compile warning telling you `InvalidInstruction` is not resolved. Let's implement it in `error.rs`.

Update dependencies:

```toml=
# Cargo.toml

...
[dependencies]
...
thiserror = "1.0.24"
```

Update `error.rs`:

```rust=
// error.rs (partially implemented)

use thiserror::Error;
use solana_program::program_error::ProgramError;

#[derive(Error, Debug, Copy, Clone)]
pub enum EscrowError {
    /// Invalid instruction
    #[error("Invalid Instruction")]
    InvalidInstruction,
    /// Not Rent Exempt
    #[error("Not Rent Exempt")]
    NotRentExempt,
}

impl From<EscrowError> for ProgramError {
    fn from(e: EscrowError) -> Self {
        ProgramError::Custom(e as u32)
    }
}
```

The main business logic locates in `processor.rs`. There will be two functions corresponding two instructions. Let's implement those one by one. Here we implement the `process_init_escrow` function which matches `EscrowInstruction::InitEscrow` case:

Update dependencies:

```toml=
# Cargo.toml

...
[dependencies]
...
spl-token = {version = "3.1.1", features = ["no-entrypoint"]}
```

Update `processor.rs`:

```rust=
// processor.rs (partially implemented)

use solana_program::{
    account_info::{next_account_info, AccountInfo},
    entrypoint::ProgramResult,
    program_error::ProgramError,
    msg,
    pubkey::Pubkey,
    program_pack::{Pack, IsInitialized},
    sysvar::{rent::Rent, Sysvar},
    program::invoke
};

use crate::{instruction::EscrowInstruction, error::EscrowError, state::Escrow};

pub struct Processor;
impl Processor {
    pub fn process(program_id: &Pubkey, accounts: &[AccountInfo], instruction_data: &[u8]) -> ProgramResult {
        let instruction = EscrowInstruction::unpack(instruction_data)?;

        match instruction {
            EscrowInstruction::InitEscrow { amount } => {
                msg!("Instruction: InitEscrow");
                Self::process_init_escrow(accounts, amount, program_id)
            }
        }
    }

    fn process_init_escrow(
        accounts: &[AccountInfo],
        amount: u64,
        program_id: &Pubkey,
    ) -> ProgramResult {
        let account_info_iter = &mut accounts.iter();
        let initializer = next_account_info(account_info_iter)?;

        if !initializer.is_signer {
            return Err(ProgramError::MissingRequiredSignature);
        }

        let temp_token_account = next_account_info(account_info_iter)?;

        let token_to_receive_account = next_account_info(account_info_iter)?;
        if *token_to_receive_account.owner != spl_token::id() {
            return Err(ProgramError::IncorrectProgramId);
        }
        
        let escrow_account = next_account_info(account_info_iter)?;
        let rent = &Rent::from_account_info(next_account_info(account_info_iter)?)?;

        if !rent.is_exempt(escrow_account.lamports(), escrow_account.data_len()) {
            return Err(EscrowError::NotRentExempt.into());
        }

        let mut escrow_info = Escrow::unpack_unchecked(&escrow_account.data.borrow())?;
        if escrow_info.is_initialized() {
            return Err(ProgramError::AccountAlreadyInitialized);
        }

        Ok(())
    }
}
```

### Part 2

You will notice a warning raised due to unresolved `state::Escrow`.

What does `state.rs` do? It basically represents the data structure stored in the account owned by Escrow program. Also, it has the pack/unpack utils to convert the data format.

Update dependencies:

```toml=
# Cargo.toml
...
[dependencies]
...
arrayref = "0.3.6"
```

Update `state.rs`:

```rust=
// state.rs

use solana_program::{
    program_pack::{IsInitialized, Pack, Sealed},
    program_error::ProgramError,
    pubkey::Pubkey,
};

use arrayref::{array_mut_ref, array_ref, array_refs, mut_array_refs};

pub struct Escrow {
    pub is_initialized: bool,
    pub initializer_pubkey: Pubkey,
    pub temp_token_account_pubkey: Pubkey,
    pub initializer_token_to_receive_account_pubkey: Pubkey,
    pub expected_amount: u64,
}

impl Sealed for Escrow {}

impl IsInitialized for Escrow {
    fn is_initialized(&self) -> bool {
        self.is_initialized
    }
}

impl Pack for Escrow {
    const LEN: usize = 105;
    fn unpack_from_slice(src: &[u8]) -> Result<Self, ProgramError> {
        let src = array_ref![src, 0, Escrow::LEN];
        let (
            is_initialized,
            initializer_pubkey,
            temp_token_account_pubkey,
            initializer_token_to_receive_account_pubkey,
            expected_amount,
        ) = array_refs![src, 1, 32, 32, 32, 8];
        let is_initialized = match is_initialized {
            [0] => false,
            [1] => true,
            _ => return Err(ProgramError::InvalidAccountData),
        };

        Ok(Escrow {
            is_initialized,
            initializer_pubkey: Pubkey::new_from_array(*initializer_pubkey),
            temp_token_account_pubkey: Pubkey::new_from_array(*temp_token_account_pubkey),
            initializer_token_to_receive_account_pubkey: Pubkey::new_from_array(*initializer_token_to_receive_account_pubkey),
            expected_amount: u64::from_le_bytes(*expected_amount),
        })
    }

    fn pack_into_slice(&self, dst: &mut [u8]) {
        let dst = array_mut_ref![dst, 0, Escrow::LEN];
        let (
            is_initialized_dst,
            initializer_pubkey_dst,
            temp_token_account_pubkey_dst,
            initializer_token_to_receive_account_pubkey_dst,
            expected_amount_dst,
        ) = mut_array_refs![dst, 1, 32, 32, 32, 8];

        let Escrow {
            is_initialized,
            initializer_pubkey,
            temp_token_account_pubkey,
            initializer_token_to_receive_account_pubkey,
            expected_amount,
        } = self;

        is_initialized_dst[0] = *is_initialized as u8;
        initializer_pubkey_dst.copy_from_slice(initializer_pubkey.as_ref());
        temp_token_account_pubkey_dst.copy_from_slice(temp_token_account_pubkey.as_ref());
        initializer_token_to_receive_account_pubkey_dst.copy_from_slice(initializer_token_to_receive_account_pubkey.as_ref());
        *expected_amount_dst = expected_amount.to_le_bytes();
    }
}
```

Let's further extend the business logic of `process_init_escrow` in `processor.rs`:

```rust=
// processor.rs (partially implemented)

...

impl Processor {
    fn process_init_escrow(
        accounts: &[AccountInfo],
        amount: u64,
        program_id: &Pubkey,
    ) -> ProgramResult {
        ...

        escrow_info.is_initialized = true;
        escrow_info.initializer_pubkey = *initializer.key;
        escrow_info.temp_token_account_pubkey = *temp_token_account.key;
        escrow_info.initializer_token_to_receive_account_pubkey = *token_to_receive_account.key;
        escrow_info.expected_amount = amount;

        Escrow::pack(escrow_info, &mut escrow_account.data.borrow_mut())?;

        let (pda, _bump_seed) = Pubkey::find_program_address(&[b"escrow"], program_id);

        let token_program = next_account_info(account_info_iter)?;
        let owner_change_ix = spl_token::instruction::set_authority(
            token_program.key,
            temp_token_account.key,
            Some(&pda),
            spl_token::instruction::AuthorityType::AccountOwner,
            initializer.key,
            &[&initializer.key],
        )?;

        msg!("Calling the token program to transfer token account ownership...");
        invoke(
            &owner_change_ix,
            &[
                temp_token_account.clone(),
                initializer.clone(),
                token_program.clone(),
            ],
        )?;

        Ok(())
    }
}
```

Here, we can see `invoke` is called to perform a CPI.

To make the first function `process_init_escrow` callable, let's put it in the `entrypoint.rs`:

```rust=
// entrypoint.rs (partially implemented)

use solana_program::{
    account_info::AccountInfo, entrypoint, entrypoint::ProgramResult, pubkey::Pubkey
};

use crate::processor::Processor;

entrypoint!(process_instruction);
fn process_instruction(
    program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8],
) -> ProgramResult {
    Processor::process(program_id, accounts, instruction_data)
}
```

Check if we can compile it successfully:

```bash
$ cargo build-bpf
...
```

### Part 3

Next, we can implement another instruction `Exchange` and its corresponding function `process_exchange`.

Update `instruction.rs`:

```rust=
// instructions.rs (fully implemented)

pub enum EscrowInstruction {
    ...

    /// Accepts a trade
    ///
    ///
    /// Accounts expected:
    ///
    /// 0. `[signer]` The account of the person taking the trade
    /// 1. `[writable]` The taker's token account for the token they send 
    /// 2. `[writable]` The taker's token account for the token they will receive should the trade go through
    /// 3. `[writable]` The PDA's temp token account to get tokens from and eventually close
    /// 4. `[writable]` The initializer's main account to send their rent fees to
    /// 5. `[writable]` The initializer's token account that will receive tokens
    /// 6. `[writable]` The escrow account holding the escrow info
    /// 7. `[]` The token program
    /// 8. `[]` The PDA account
    Exchange {
        /// the amount the taker expects to be paid in the other token, as a u64 because that's the max possible supply of a token
        amount: u64,
    }
}

impl EscrowInstruction {
    ...

    pub fn unpack(input: &[u8]) -> Result<Self, ProgramError> {
        ...

        Ok(match tag {
            ...
            1 => Self::Exchange {
                amount: Self::unpack_amount(rest)?
            },
            ...
        })
    }
}
```

Also in `processor.rs`:

```rust=
// processor.rs (fully implemented)

use solana_program::{
    account_info::{next_account_info, AccountInfo},
    entrypoint::ProgramResult,
    msg,
    program::{invoke, invoke_signed},
    program_error::ProgramError,
    program_pack::{IsInitialized, Pack},
    pubkey::Pubkey,
    sysvar::{rent::Rent, Sysvar},
};

use spl_token::state::Account as TokenAccount;

...

impl Processor {
    pub fn process(
        program_id: &Pubkey,
        accounts: &[AccountInfo],
        instruction_data: &[u8],
    ) -> ProgramResult {
        ...

        match instruction {
            ...

            EscrowInstruction::Exchange { amount } => {
                msg!("Instruction: Exchange");
                Self::process_exchange(accounts, amount, program_id)
            }
        }
    }

    fn process_exchange(
        accounts: &[AccountInfo],
        amount_expected_by_taker: u64,
        program_id: &Pubkey,
    ) -> ProgramResult {
        let account_info_iter = &mut accounts.iter();
        let taker = next_account_info(account_info_iter)?;

        if !taker.is_signer {
            return Err(ProgramError::MissingRequiredSignature);
        }

        let takers_sending_token_account = next_account_info(account_info_iter)?;

        let takers_token_to_receive_account = next_account_info(account_info_iter)?;

        let pdas_temp_token_account = next_account_info(account_info_iter)?;
        let pdas_temp_token_account_info =
            TokenAccount::unpack(&pdas_temp_token_account.data.borrow())?;
        let (pda, bump_seed) = Pubkey::find_program_address(&[b"escrow"], program_id);

        if amount_expected_by_taker != pdas_temp_token_account_info.amount {
            return Err(EscrowError::ExpectedAmountMismatch.into());
        }

        let initializers_main_account = next_account_info(account_info_iter)?;
        let initializers_token_to_receive_account = next_account_info(account_info_iter)?;
        let escrow_account = next_account_info(account_info_iter)?;

        let escrow_info = Escrow::unpack(&escrow_account.data.borrow())?;

        if escrow_info.temp_token_account_pubkey != *pdas_temp_token_account.key {
            return Err(ProgramError::InvalidAccountData);
        }

        if escrow_info.initializer_pubkey != *initializers_main_account.key {
            return Err(ProgramError::InvalidAccountData);
        }

        if escrow_info.initializer_token_to_receive_account_pubkey != *initializers_token_to_receive_account.key {
            return Err(ProgramError::InvalidAccountData);
        }

        let token_program = next_account_info(account_info_iter)?;

        let transfer_to_initializer_ix = spl_token::instruction::transfer(
            token_program.key,
            takers_sending_token_account.key,
            initializers_token_to_receive_account.key,
            taker.key,
            &[&taker.key],
            escrow_info.expected_amount,
        )?;
        msg!("Calling the token program to transfer tokens to the escrow's initializer...");
        invoke(
            &transfer_to_initializer_ix,
            &[
                takers_sending_token_account.clone(),
                initializers_token_to_receive_account.clone(),
                taker.clone(),
                token_program.clone(),
            ],
        )?;

        let pda_account = next_account_info(account_info_iter)?;

        let transfer_to_taker_ix = spl_token::instruction::transfer(
            token_program.key,
            pdas_temp_token_account.key,
            takers_token_to_receive_account.key,
            &pda,
            &[&pda],
            pdas_temp_token_account_info.amount,
        )?;
        msg!("Calling the token program to transfer tokens to the taker...");
        invoke_signed(
            &transfer_to_taker_ix,
            &[
                pdas_temp_token_account.clone(),
                takers_token_to_receive_account.clone(),
                pda_account.clone(),
                token_program.clone(),
            ],
            &[&[&b"escrow"[..], &[bump_seed]]],
        )?;

        let close_pdas_temp_acc_ix = spl_token::instruction::close_account(
            token_program.key,
            pdas_temp_token_account.key,
            initializers_main_account.key,
            &pda,
            &[&pda]
        )?;
        msg!("Calling the token program to close pda's temp account...");
        invoke_signed(
            &close_pdas_temp_acc_ix,
            &[
                pdas_temp_token_account.clone(),
                initializers_main_account.clone(),
                pda_account.clone(),
                token_program.clone(),
            ],
            &[&[&b"escrow"[..], &[bump_seed]]],
        )?;

        msg!("Closing the escrow account...");
        **initializers_main_account.lamports.borrow_mut() = initializers_main_account.lamports()
        .checked_add(escrow_account.lamports())
        .ok_or(EscrowError::AmountOverflow)?;
        **escrow_account.lamports.borrow_mut() = 0;
        *escrow_account.data.borrow_mut() = &mut [];

        Ok(())
    }
}
```

Here we can see that `invoke_signed` is called with seeds since the owner of escrow account is a PDA.

Finally, implement the missing error enums:

```rust=
// error.rs (fully implemented)

...

pub enum EscrowError {
    ...

    /// Expected Amount Mismatch
    #[error("Expected Amount Mismatch")]
    ExpectedAmountMismatch,
    /// Amount Overflow
    #[error("Amount Overflow")]
    AmountOverflow,
}
```

Check if we can compile successfully:

```bash
$ cargo build-bpf
...
```

### Interact with the escrow program

- See [this repo](https://github.com/paul-schaaf/solana-escrow/tree/master/scripts) for more details

#### Basic setup

Now, we can write some client side code to interact with the escrow program.

First, let's install dependencies:

```bash
$ npm init -y
...

$ npm install --save @solana/spl-token @solana/web3.js bn.js
...

$ tsc --init
...
```

Next, let's generate the files to be filled in necessary code and data:

```
$ mkdir keys
$ touch keys/id_pub.json
$ touch keys/alice_pub.json
$ touch keys/bob_pub.json
$ touch keys/program_pub.json

$ mkdir ts
$ touch ts/setup.ts
$ touch ts/utils.ts
$ touch ts/alice.ts
$ touch ts/bob.ts

$ touch terms.json
```

#### Generate Keypairs

We have to generate keypairs for `alice`, `bob`, and the transaction `payer`. This can be done via `solana-keygen`:

```bash
$ solana-keygen new -o keys/id.json
...

$ solana-keygen new -o keys/alice.json
...

$ solana-keygen new -o keys/bob.json
...
```

Next, we need to manually update the public keys for each. Retrieve the address for **all of them** and paste it to the `*_pub.json` files accordingly. For example:

```bash
$ solana address -k keys/id.json
9q9XLUDjDKj2cahaN44X9Mid2HGJtUauFvjJG8qocY5a
```

```json=
// id_pub.json

"9q9XLUDjDKj2cahaN44X9Mid2HGJtUauFvjJG8qocY5a"
```

> Don't forget the double quotes

#### Add Code Base

Here we add the client code base. Copy and paste the following files to your local code base:
- [`ts/setup.ts`](https://github.com/paul-schaaf/solana-escrow/blob/master/scripts/src/setup.ts)
- [`ts/utils`](https://github.com/paul-schaaf/solana-escrow/blob/master/scripts/src/utils.ts)
- [`ts/alice.ts`](https://github.com/paul-schaaf/solana-escrow/blob/master/scripts/src/alice.ts)
- [`ts/bob.ts`](https://github.com/paul-schaaf/solana-escrow/blob/master/scripts/src/bob.ts)

> Again, I strongly recommend you to clone the original code base and run it

#### Compile, Depoly and Setup

First, let's start the validator:

```bash
$ solana-test-validator
...
```

Compile and deploy the program:

```bash
$ cargo build-bpf
...

$ solana program deploy target/deploy/solana_escrow.so
Program Id: EKnr6pssVnPmoGJH3NgtCByF9jMDRnyDQZxkHqz1GBS2
```

Before we execute the client code, we need to update the `programId` to be looked up:

```json=
// program_pub.json

"EKnr6pssVnPmoGJH3NgtCByF9jMDRnyDQZxkHqz1GBS2"
```

Also, update the predefined `terms.json` as follows:

```json=
// terms.json

{
  "aliceExpectedAmount": 3,
  "bobExpectedAmount": 5
}
```

Fund the transaction `payer` in advance:

```bash
$ solana transfer 9q9XLUDjDKj2cahaN44X9Mid2HGJtUauFvjJG8qocY5a 100 --allow-unfunded-recipient
```

#### Run the Client Code

Finally, let's run the client code:

First, run `setup.ts` to mint the tokens to be exchanged:

```bash
$ ts-node ts/setup.ts
...
```

Next, run `alice.ts` to initialize the escrow program:

```bash
$ ts-node ts/alice.ts
...
```

You can see how an instruction is constructed. The interger `0` assined to the `Uint8Array` represents the instruction `InitEscrow`:

```typescript=
// alice.ts

...

const alice = async () => {
  const initEscrowIx = new TransactionInstruction({
    programId: escrowProgramId,
    keys: [
      { pubkey: aliceKeypair.publicKey, isSigner: true, isWritable: false },
      {
        pubkey: tempXTokenAccountKeypair.publicKey,
        isSigner: false,
        isWritable: true,
      },
      {
        pubkey: aliceYTokenAccountPubkey,
        isSigner: false,
        isWritable: false,
      },
      { pubkey: escrowKeypair.publicKey, isSigner: false, isWritable: true },
      { pubkey: SYSVAR_RENT_PUBKEY, isSigner: false, isWritable: false },
      { pubkey: TOKEN_PROGRAM_ID, isSigner: false, isWritable: false },
    ],
    data: Buffer.from(
      Uint8Array.of(0, ...new BN(terms.aliceExpectedAmount).toArray("le", 8))
    ),
  });

  ...
}
```

Then, run `bob.ts` to exchange and close the escrow account:

```bash
$ ts-node ts/bob.ts
...
```

## 3. Escrow Program (using Anchor)

### Goal

- Learn the best practice
- Why use Anchor?
    - Remove Boilerplate
    - Make Solana program [safer](https://twitter.com/armaniferrante/status/1411589634228772870)
    - Clearer code structure
- A good framework reduces the mental pressure and keep the precious attention resource to important things
- **I actually wrote another post explaining the whole thing.** See [this doc](https://hackmd.io/@ironaddicteddog/solana-anchor-escrow) to learn more.

## More Advanced Topics

- Open-sourced Projects
    - [Serum](https://github.com/project-serum)
    - [Raydium](https://github.com/raydium-io)
    - [Saber](https://github.com/saber-hq)
    - ...
- Solana Program Library
    - [`token` Program](https://github.com/solana-labs/solana-program-library/tree/master/token)
    - [`token-swap` Program](https://github.com/solana-labs/solana-program-library/tree/master/token-swap)
    - [Associated Token Account](https://spl.solana.com/associated-token-account)
    - ...
- [Anchor AMM](https://github.com/ironaddicteddog/anchor-amm)

## References

### General

- https://medium.com/@asmiller1989/solana-transactions-in-depth-1f7f7fe06ac2 
- https://hackmd.io/@adamisrusty/HkVyZHBoO
- https://2501babe.github.io/posts/solana101.html
- https://github.com/paul-schaaf/awesome-solana
- https://github.com/project-serum/awesome-serum
- https://solana.com/developers

### Front-End Development

- https://github.com/yihau/full-stack-solana-development
- https://github.com/yihau/solana-web3-demo
- https://github.com/raydium-io/raydium-ui
- https://github.com/thuglabs/create-dapp-solana-nextjs

### Program Development

- https://paulx.dev/blog/2021/01/14/programming-on-solana-an-introduction/#instruction-rs-part-1-general-code-structure-and-the-beginning-of-the-escrow-program-flow
- https://github.com/jstarry/solana-workshop-tw
- https://jstarry.notion.site/Program-deploys-29780c48794c47308d5f138074dd9838
- https://jstarry.notion.site/Transaction-Fees-f09387e6a8d84287aa16a34ecb58e239

### Anchor Tutorials

- https://hackmd.io/@ironaddicteddog/anchor_example_escrow
- https://github.com/ironaddicteddog/anchor-escrow
- https://github.com/ironaddicteddog/anchor-amm
- https://dev.to/dabit3/the-complete-guide-to-full-stack-solana-development-with-react-anchor-rust-and-phantom-3291
- https://2501babe.github.io/posts/anchor101.html
- https://www.brianfriel.xyz/learning-how-to-build-on-solana/
- https://project-serum.github.io/anchor/tutorials/tutorial-0.html

### Core Technology

- https://medium.com/solana-labs/proof-of-history-explained-by-a-water-clock-e682183417b8
- https://medium.com/solana-labs/proof-of-history-a-clock-for-blockchain-cf47a61a9274
- https://medium.com/solana-labs/sealevel-parallel-processing-thousands-of-smart-contracts-d814b378192
- https://medium.com/solana-labs/solanas-network-architecture-8e913e1d5a40
- https://medium.com/solana-labs/7-innovations-that-make-solana-the-first-web-scale-blockchain-ddc50b1defda
- https://jito-labs.medium.com/solana-validator-101-transaction-processing-90bcdc271143

### Twitters

- https://twitter.com/ironaddicteddog
- https://twitter.com/armaniferrante
- https://twitter.com/therealchaseeb
- https://twitter.com/jstrry

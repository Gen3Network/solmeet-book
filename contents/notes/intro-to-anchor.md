---
tags: notes
---

# Anchor: A Powerful Framework for Solana Developers

**Author:** [@ironaddicteddog](https://twitter.com/ironaddicteddog)

***[Updated at 2022.3.31]***

> You can find the full code base [here](https://github.com/ironaddicteddog/anchor-escrow)

## What is Anchor?

There is a comprehensive explanation on the [official website](https://project-serum.github.io/anchor/getting-started/introduction.html). Let me just quote relative paragraphs here:

> Anchor is a framework for Solana's Sealevel runtime providing several convenient developer tools.
> 
> If you're familiar with developing in Ethereum's Solidity, Truffle, web3.js, then the experience will be familiar. Although the DSL syntax and semantics are targeted at Solana, the high level flow of writing RPC request handlers, emitting an IDL, and generating clients from IDL is the same.

In short, Anchor gives you the following handy tools for developing Solana programs:

- **Rust crates and eDSL for writing Solana programs**
- **IDL specification**
- **TypeScript package for generating clients from IDL**
- **CLI and workspace management for developing complete applications**

You can watch [this awesome talk](https://youtu.be/cvW8EwGHw8U) given by Armani Ferrante at Breakpoint 2021 to feel the power of Anchor.

### Workflow

![](https://i.imgur.com/jkObSKO.jpg)

1. Develop the **program** (Smart Contract)
2. Build the program and export the **IDL**
3. Generate the **client** representation of program from the IDL to interact with the program

### Why Anchor?

- Productivity
    - Make Solana program more intuitive to understand
    - More clear buisness Logic
    - Remove a ton of biolderplate code
- Security
    - Customized Account Validation
        - Singer
        - Mut
        - ...
    - Discriminator
      - Discriminator is generated and inserted into the first 8 bytes of account data. Ex: `sha256("account:<MyAccountName>")[..8] || borsh(account_struct)`
      - Used for more secure account validation and function dispatch
      - See [this Twitter thread](https://twitter.com/armaniferrante/status/1411589634228772870) for more details
      - See [here](https://github.com/project-serum/anchor/blob/master/ts/src/program/namespace/index.ts#L53) and [here](https://github.com/project-serum/anchor/blob/master/lang/syn/src/codegen/program/dispatch.rs#L146) for the actual implementation

## Before We Start

### Why Rust? Why Solana?

You can refer to this [doc](https://hackmd.io/@ironaddicteddog/solana-starter-kit#Before-We-Start) for the motivations.

### Prerequisites

- [Solana Helloworld](https://hackmd.io/@ironaddicteddog/solana-starter-kit#1-Hello-World)
- [**Solana Escrow Program (using vanilla Rust)**](https://hackmd.io/@ironaddicteddog/solana-starter-kit#2-Escrow-Program-using-vanilla-Rust)
- [Anchor Basic Tutorials](https://project-serum.github.io/anchor/tutorials/tutorial-0.html)

## Installation

Install `avm`:

```bash
$ cargo install --git https://github.com/project-serum/anchor avm --locked --force
...
```

Install latest `anchor` version:

```bash
$ avm install latest
...
$ avm use latest
...
```

> If you haven't installed `cargo`, please refer to this [doc](https://hackmd.io/@ironaddicteddog/solana-starter-kit#Install-Rust-and-Solana-Cli) for installation steps.

### Extra Dependencies on Linux (Optional)

You may have to install some extra dependencies on Linux (ex. Ubuntu):

```bash
$ sudo apt-get update && sudo apt-get upgrade && sudo apt-get install -y pkg-config build-essential libudev-dev
...
```

### Verify the Installation

Check if Anchor is successfully installed:

```bash
$ anchor --version
anchor-cli 0.22.0
```

## Escrow Program

> Reminder: you can find the full code base for this example [here](https://github.com/ironaddicteddog/anchor-escrow). However, I would strongly recommend you to go through the copy-paste with me to get familiar with the flow.

Next, let's develop an escrow program using Anchor. I strongly recommend you to go through [this tutorial](https://hackmd.io/@ironaddicteddog/solana-starter-kit#2-Escrow-Program-using-vanilla-Rust) if you are not familiar with escrow program yet.

### Overview

Since this program is extended from the original [Escrow Program](https://github.com/paul-schaaf/solana-escrow), I assumed you have gone through the [original blog post](https://paulx.dev/blog/2021/01/14/programming-on-solana-an-introduction/#instruction-rs-part-1-general-code-structure-and-the-beginning-of-the-escrow-program-flow) at least once.

However, there is one major difference between this exmaple and the original Escrow program: Instead of letting initializer create a token account to be reset to a PDA authority, we create a token account `Vault` that has both a PDA key and a PDA authority.

#### Initialize

![](https://i.imgur.com/VmRKZUy.png)

`Initializer` can send a transaction to the escrow program to initialize the Vault. In this transaction, two new accounts: `Vault` and `EscrowAccount`, will be created and tokens (Token A) to be exchanged will be transfered from `Initializer` to `Vault`.

#### Cancel

![](https://i.imgur.com/f6ahGXy.png)

`Initializer` can also send a transaction to the escrow program to cancel the demand of escrow. The tokens will be transfered back to the `Initialzer` and both `Vault` and `EscrowAccount` will be closed in this case.

#### Exchange

![](https://i.imgur.com/MzG26dm.png)

`Taker` can send a transaction to the escrow to exchange Token B for Token A. First, tokens (Token B) will be transfered from `Taker` to `Initializer`. Afterward, the tokens (Token A) kept in the Vault will be transfered to `Taker`. Finally, both `Vault` and `EscrowAccount` will be closed.


### Initialize the Program

First, let's start a fresh Anchor project:

```bash
$ anchor init anchor-escrow
...
```

This handy command will populate a project folder including the following files:
- `Cargo.toml`
- `Anchor.toml`
- `package.json`
- `tsconfig.json`
- ...

### Program Architecture

There are 3 main parts in the program:
- **Processor**: Main buisiness logic locates in processor
- **Account Context (Instructions)**: Instruction data packing/unpacking and account constraints and access control locate in Instruction handling part
- **Account**: Declaration of account owned by program locates in account part

### Dependencies

Before we dive into the program, we need add the missing dependencies in `Cargo.toml`:

```toml
# Cargo.toml

...
[dependencies]
anchor-lang = "0.20.1"
anchor-spl = {version = "0.20.1"}
spl-token = {version = "3.3.0", features = ["no-entrypoint"]}
```

### Update `program_id` (Optional)

There is a default `program_id` defined by `declare_id!` macro in `lib.rs`:

```rust
// lib.rs

...

declare_id!("Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS");

...
```

Although we can use the default value just fine, I would strongly recommend to replace this with the actual `program_id`, which is the public key of the deploy key.

Get the public key of the deploy key:

```bash
$ anchor keys list
anchor_escrow: Hfd7V12kj9AENQjLpTozaPW6aT2rhPm3LSyjXZ5AbWH
```

Replace the default value of `program_id` with this new value:

```toml
# Anchor.toml

[programs.localnet]
anchor_escrow = "Hfd7V12kj9AENQjLpTozaPW6aT2rhPm3LSyjXZ5AbWH"

...
```

```rust
// lib.rs

...

declare_id!("Hfd7V12kj9AENQjLpTozaPW6aT2rhPm3LSyjXZ5AbWH");

...
```

### Processor (Part 1)

Let's scaffold the processor first. There should be 3 functions corresponding 3 tasks listed above:

```rust=
// Processor (unimplemented)

#[program]
pub mod anchor_escrow {
    use super::*;

    pub fn initialize(
        ctx: Context<Initialize>,
        _vault_account_bump: u8,
        initializer_amount: u64,
        taker_amount: u64,
    ) -> ProgramResult {
        // TODO
        Ok(())
    }

    pub fn cancel(ctx: Context<Cancel>) -> ProgramResult {
        // TODO
        Ok(())
    }

    pub fn exchange(ctx: Context<Exchange>) -> ProgramResult {
        // TODO
        Ok(())
    }
}
```

The `#[program]` keyword is what makes the magic happen. In argument `ctx`, notice that we have to use a type `Initialize` for `Context<T>` generic. `Initialize` can be considered as a wrapper for instructions. This wrapper is enhanced by Anchor via derived macro (`#[derive(account)]`). We will see how it works real quick.

Each function has a corresponding instruction. As a result, there will be 3 instruction wrappers.

<!-- > rust-analyzer does not warn. Why? -->

### Instructions (Part 1)

From the processor section, we know that each function defined needs a corresponding instruction. So let's define those in instruction section:

```rust=
// Instructions (unimplemented)

#[derive(Accounts)]
pub struct Initialize<'info> {
    // TODO
}

#[derive(Accounts)]
pub struct Exchange<'info> {
    // TODO
}

#[derive(Accounts)]
pub struct Cancel<'info> {
    // TODO
}
```

Depending on the program functions, the instructions should bring in the accounts that are needed for operations.

To see what are accounts needed for initializing escrow account, we have to consider what data stored in escrow account first.

### Program Account

Accounts that are owned and managed by the program are defined in the `#[account]` section.

<!-- - What is the difference between the instruction section and account section
- What state should be managed? The metadata needed for the integrity / validation of the escrow program
 -->

#### `EscrowAccount`

| Field | Type | Description |
| - | - | - |
| `initializer_key` | `Pubkey` | To authorize the actions properly |
| `initializer_deposit_token_account` | `Pubkey` | To record the deposit account of initialzer |
| `initializer_receive_token_account` | `Pubkey` | To record the receiving account of initializer |
| `initializer_amount` | `u64` | To record how much token should the initializer transfer to taker |
| `taker_amount` | `u64` | To record how much token should the initializer receive from the taker |

As a result, we should design an account that stores the minimum information to validate the escrow state and keep the integrity of the program:

```rust=
// Program Account (fully implemented)

#[account]
pub struct EscrowAccount {
    pub initializer_key: Pubkey,
    pub initializer_deposit_token_account: Pubkey,
    pub initializer_receive_token_account: Pubkey,
    pub initializer_amount: u64,
    pub taker_amount: u64,
}
```

### Instructions (Part 2)

According to what we have in `EscrowAccount`, we need the following accounts to initialize it.

#### `Initialize` 

| Field | Type | Description |
| - | - | - |
| **initializer** | `AccountInfo` | Signer of `InitialEscrow` instruction. To be stored in `EscrowAccount` |
| **initializer_deposit_token_account** | `Account<TokenAccount>` | The account of token account for token exchange. To be stored in `EscrowAccount` |
| **initializer_receive_token_account** | `Account<TokenAccount>` | The account of token account for token exchange. To be stored in `EscrowAccount` |
| **token_program** | `AccountInfo` | The account of `TokenProgram` |
| **escrow_account** | `Box<Account<EscrowAccount>>` | The account of `EscrowAccount` |
| **vault_account** | `Account<TokenAccount>` | The account of `Vault`, which is created by Anchor via **constraints**. (Will be explained in part 3) |
| **mint** | `Account<Mint>` | - |
| **system_program** | `AccountInfo` | - |
| **rent** | `Sysvar<Rent>` |  - |

#### `Cancel`

| Field | Type | Description |
| - | - | - |
| **initializer** | `AccountInfo` | The initializer of `EscrowAccount` |
| **initializer_deposit_token_account** | `Account<TokenAccount>` | The address of token account for token exchange |
| **vault_account** | `Account<TokenAccount>` | The program derived address |
| **vault_authority** | `AccountInfo` | The program derived address |
| **escrow_account** | `Box<Account<EscrowAccount>>` | The address of `EscrowAccount`. Have to check if the `EscrowAccount` follows certain constraints. |
| **token_program** | `AccountInfo` | The address of `TokenProgram` |

#### `Exchange`

| Field | Type | Description |
| - | - | - |
| **taker** | `AccountInfo` | Singer of `Exchange` instruction |
| **taker_deposit_token_account** | `Account<TokenAccount>` | Token account for token exchange |
| **taker_receive_token_account** | `Account<TokenAccount>` | Token account for token exchange |
| **initializer_deposit_token_account** | `Account<TokenAccount>` | Token account for token exchange |
| **initializer_receive_token_account** | `Account<TokenAccount>` | Token account for token exchange |
| **initializer** | `AccountInfo` | To be used in **constraints**. (Will explain in part 3) |
| **escrow_account** | `Box<Account<EscrowAccount>>` | The address of `EscrowAccount`. Have to check if the `EscrowAccount` follows certain constraints. |
| **vault_account** | `Account<TokenAccount>` | The program derived address |
| **vault_authority** | `AccountInfo` | The program derived address |
| **token_program** | `AccountInfo` | The address of `TokenProgram` |

You can tell this is a very long list of inputs since Solana programs are **stateless**.

```rust=
// Instructions (partially implemented)

use anchor_spl::token::{self, CloseAccount, Mint, SetAuthority, TokenAccount, Transfer};
use spl_token::instruction::AuthorityType;
...

#[derive(Accounts)]
pub struct Initialize<'info> {
    pub initializer: AccountInfo<'info>,
    pub mint: Account<'info, Mint>,
    pub vault_account: Account<'info, TokenAccount>,
    pub initializer_deposit_token_account: Account<'info, TokenAccount>,
    pub initializer_receive_token_account: Account<'info, TokenAccount>,
    pub escrow_account: Box<Account<'info, EscrowAccount>>,
    pub system_program: AccountInfo<'info>,
    pub rent: Sysvar<'info, Rent>,
    pub token_program: AccountInfo<'info>,
}

#[derive(Accounts)]
pub struct Cancel<'info> {
    pub initializer: AccountInfo<'info>,
    pub initializer_deposit_token_account: Account<'info, TokenAccount>,
    pub vault_account: Account<'info, TokenAccount>,
    pub vault_authority: AccountInfo<'info>,
    pub escrow_account: Box<Account<'info, EscrowAccount>>,
    pub token_program: AccountInfo<'info>,
}

#[derive(Accounts)]
pub struct Exchange<'info> {
    pub taker: AccountInfo<'info>,
    pub taker_deposit_token_account: Account<'info, TokenAccount>,
    pub taker_receive_token_account: Account<'info, TokenAccount>,
    pub initializer_deposit_token_account: Account<'info, TokenAccount>,
    pub initializer_receive_token_account: Account<'info, TokenAccount>,
    pub initializer: AccountInfo<'info>,
    pub escrow_account: Box<Account<'info, EscrowAccount>>,
    pub vault_account: Account<'info, TokenAccount>,
    pub vault_authority: AccountInfo<'info>,
    pub token_program: AccountInfo<'info>,
}
```

> Notice the lifetime anotation used in generic

You can see there are 2 different types for account: `AccountInfo` and `Account`. So what is the difference? I suppose it's proper to use `Account` over `AccountInfo` when you want Anchor to deserialize the data for convenience. In that case, you can access the account data via a trivial method call. For example: `ctx.accounts.vault_account.mint`

### Processor (Part 2)

With necessary accounts, we can implement the business logic inside processor without bothering:

```rust=
// Processor (fully implenmented)

#[program]
pub mod anchor_escrow {
    use super::*;

    const ESCROW_PDA_SEED: &[u8] = b"escrow";

    pub fn initialize(
        ctx: Context<Initialize>,
        _vault_account_bump: u8,
        initializer_amount: u64,
        taker_amount: u64,
    ) -> ProgramResult {
        ctx.accounts.escrow_account.initializer_key = *ctx.accounts.initializer.key;
        ctx.accounts
            .escrow_account
            .initializer_deposit_token_account = *ctx
            .accounts
            .initializer_deposit_token_account
            .to_account_info()
            .key;
        ctx.accounts
            .escrow_account
            .initializer_receive_token_account = *ctx
            .accounts
            .initializer_receive_token_account
            .to_account_info()
            .key;
        ctx.accounts.escrow_account.initializer_amount = initializer_amount;
        ctx.accounts.escrow_account.taker_amount = taker_amount;

        let (vault_authority, _vault_authority_bump) =
            Pubkey::find_program_address(&[ESCROW_PDA_SEED], ctx.program_id);
        token::set_authority(
            ctx.accounts.into_set_authority_context(),
            AuthorityType::AccountOwner,
            Some(vault_authority),
        )?;

        token::transfer(
            ctx.accounts.into_transfer_to_pda_context(),
            ctx.accounts.escrow_account.initializer_amount,
        )?;

        Ok(())
    }

    pub fn cancel(ctx: Context<Cancel>) -> ProgramResult {
        let (_vault_authority, vault_authority_bump) =
            Pubkey::find_program_address(&[ESCROW_PDA_SEED], ctx.program_id);
        let authority_seeds = &[&ESCROW_PDA_SEED[..], &[vault_authority_bump]];

        token::transfer(
            ctx.accounts
                .into_transfer_to_initializer_context()
                .with_signer(&[&authority_seeds[..]]),
            ctx.accounts.escrow_account.initializer_amount,
        )?;

        token::close_account(
            ctx.accounts
                .into_close_context()
                .with_signer(&[&authority_seeds[..]]),
        )?;

        Ok(())
    }

    pub fn exchange(ctx: Context<Exchange>) -> ProgramResult {
        let (_vault_authority, vault_authority_bump) =
            Pubkey::find_program_address(&[ESCROW_PDA_SEED], ctx.program_id);
        let authority_seeds = &[&ESCROW_PDA_SEED[..], &[vault_authority_bump]];

        token::transfer(
            ctx.accounts.into_transfer_to_initializer_context(),
            ctx.accounts.escrow_account.taker_amount,
        )?;

        token::transfer(
            ctx.accounts
                .into_transfer_to_taker_context()
                .with_signer(&[&authority_seeds[..]]),
            ctx.accounts.escrow_account.initializer_amount,
        )?;

        token::close_account(
            ctx.accounts
                .into_close_context()
                .with_signer(&[&authority_seeds[..]]),
        )?;

        Ok(())
    }
}
```

Now the business logic is simple, straightforward, and clear to understand.

- In `initialize`, what happens is that the input accounts are assigned to `EscrowAccount` fields one by one. Then, a program derived address, or PDA, is derived to be going to become new authority of `initializer_deposit_token_account`.
- In `cancel`, it just simply reset the authority from PDA back to the initializer.
- In `exchange`, 3 things happen:
    - First, token A gets transfered from `pda_deposit_token_account` to `taker_receive_token_account`.
    - Next, token B gets transfered from `taker_deposit_token_account` to `initializer_receive_token_account`.
    - Finally, the authority of `pda_deposit_token_account` gets set back to the `initializer`.

### Utils

There are some util functions used for wrapping the data to be passed in `tokens::transfer`, `token::close_account` and `token::set_authority`. It might look a bit overwhelmed in the first place. However, the purpose behind these functions are clear and simple:

```rust=
// Utils (fully implemented)

impl<'info> Initialize<'info> {
    fn into_transfer_to_pda_context(&self) -> CpiContext<'_, '_, '_, 'info, Transfer<'info>> {
        let cpi_accounts = Transfer {
            from: self
                .initializer_deposit_token_account
                .to_account_info()
                .clone(),
            to: self.vault_account.to_account_info().clone(),
            authority: self.initializer.clone(),
        };
        CpiContext::new(self.token_program.clone(), cpi_accounts)
    }

    fn into_set_authority_context(&self) -> CpiContext<'_, '_, '_, 'info, SetAuthority<'info>> {
        let cpi_accounts = SetAuthority {
            account_or_mint: self.vault_account.to_account_info().clone(),
            current_authority: self.initializer.clone(),
        };
        CpiContext::new(self.token_program.clone(), cpi_accounts)
    }
}

impl<'info> Cancel<'info> {
    fn into_transfer_to_initializer_context(
        &self,
    ) -> CpiContext<'_, '_, '_, 'info, Transfer<'info>> {
        let cpi_accounts = Transfer {
            from: self.vault_account.to_account_info().clone(),
            to: self
                .initializer_deposit_token_account
                .to_account_info()
                .clone(),
            authority: self.vault_authority.clone(),
        };
        CpiContext::new(self.token_program.clone(), cpi_accounts)
    }

    fn into_close_context(&self) -> CpiContext<'_, '_, '_, 'info, CloseAccount<'info>> {
        let cpi_accounts = CloseAccount {
            account: self.vault_account.to_account_info().clone(),
            destination: self.initializer.clone(),
            authority: self.vault_authority.clone(),
        };
        CpiContext::new(self.token_program.clone(), cpi_accounts)
    }
}

impl<'info> Exchange<'info> {
    fn into_transfer_to_initializer_context(
        &self,
    ) -> CpiContext<'_, '_, '_, 'info, Transfer<'info>> {
        let cpi_accounts = Transfer {
            from: self.taker_deposit_token_account.to_account_info().clone(),
            to: self
                .initializer_receive_token_account
                .to_account_info()
                .clone(),
            authority: self.taker.clone(),
        };
        CpiContext::new(self.token_program.clone(), cpi_accounts)
    }

    fn into_transfer_to_taker_context(&self) -> CpiContext<'_, '_, '_, 'info, Transfer<'info>> {
        let cpi_accounts = Transfer {
            from: self.vault_account.to_account_info().clone(),
            to: self.taker_receive_token_account.to_account_info().clone(),
            authority: self.vault_authority.clone(),
        };
        CpiContext::new(self.token_program.clone(), cpi_accounts)
    }

    fn into_close_context(&self) -> CpiContext<'_, '_, '_, 'info, CloseAccount<'info>> {
        let cpi_accounts = CloseAccount {
            account: self.vault_account.to_account_info().clone(),
            destination: self.initializer.clone(),
            authority: self.vault_authority.clone(),
        };
        CpiContext::new(self.token_program.clone(), cpi_accounts)
    }
}
```

### Instructions (Part 3)

Finally, let's talk about the account constraints. Here comes a very handy funcionality that Anchor provides: Account Constraints.

Constraints are useful for basic checkings such as whether the initializer is the signer of instruction.

> If you are familiar of Solidity, you can map this concept to solidity modifier.

```rust=
// Instructions (fully implementated)

#[derive(Accounts)]
#[instruction(vault_account_bump: u8, initializer_amount: u64)]
pub struct Initialize<'info> {
    #[account(mut, signer)]
    pub initializer: AccountInfo<'info>,
    pub mint: Account<'info, Mint>,
    #[account(
        init,
        seeds = [b"token-seed".as_ref()],
        bump = vault_account_bump,
        payer = initializer,
        token::mint = mint,
        token::authority = initializer,
    )]
    pub vault_account: Account<'info, TokenAccount>,
    #[account(
        mut,
        constraint = initializer_deposit_token_account.amount >= initializer_amount
    )]
    pub initializer_deposit_token_account: Account<'info, TokenAccount>,
    pub initializer_receive_token_account: Account<'info, TokenAccount>,
    #[account(zero)]
    pub escrow_account: Box<Account<'info, EscrowAccount>>,
    pub system_program: AccountInfo<'info>,
    pub rent: Sysvar<'info, Rent>,
    pub token_program: AccountInfo<'info>,
}

#[derive(Accounts)]
pub struct Cancel<'info> {
    #[account(mut, signer)]
    pub initializer: AccountInfo<'info>,
    #[account(mut)]
    pub vault_account: Account<'info, TokenAccount>,
    pub vault_authority: AccountInfo<'info>,
    #[account(mut)]
    pub initializer_deposit_token_account: Account<'info, TokenAccount>,
    #[account(
        mut,
        constraint = escrow_account.initializer_key == *initializer.key,
        constraint = escrow_account.initializer_deposit_token_account == *initializer_deposit_token_account.to_account_info().key,
        close = initializer
    )]
    pub escrow_account: Box<Account<'info, EscrowAccount>>,
    pub token_program: AccountInfo<'info>,
}

#[derive(Accounts)]
pub struct Exchange<'info> {
    #[account(signer)]
    pub taker: AccountInfo<'info>,
    #[account(mut)]
    pub taker_deposit_token_account: Account<'info, TokenAccount>,
    #[account(mut)]
    pub taker_receive_token_account: Account<'info, TokenAccount>,
    #[account(mut)]
    pub initializer_deposit_token_account: Account<'info, TokenAccount>,
    #[account(mut)]
    pub initializer_receive_token_account: Account<'info, TokenAccount>,
    #[account(mut)]
    pub initializer: AccountInfo<'info>,
    #[account(
        mut,
        constraint = escrow_account.taker_amount <= taker_deposit_token_account.amount,
        constraint = escrow_account.initializer_deposit_token_account == *initializer_deposit_token_account.to_account_info().key,
        constraint = escrow_account.initializer_receive_token_account == *initializer_receive_token_account.to_account_info().key,
        constraint = escrow_account.initializer_key == *initializer.key,
        close = initializer
    )]
    pub escrow_account: Box<Account<'info, EscrowAccount>>,
    #[account(mut)]
    pub vault_account: Account<'info, TokenAccount>,
    pub vault_authority: AccountInfo<'info>,
    pub token_program: AccountInfo<'info>,
}
```

Here, we can see a few new attributes, such as:

| Attribute | Description |
| - | - |
| `#[account(signer)]` | Checks the given account signed the transaction |
| `#[account(mut)]` | Marks the account as mutable and persists the state transition |
| `#[account(constraint = <expression\>)]` | Executes the given code as a constraint. The expression should evaluate to a boolean |
| `#[account(close = <target\>)]` | Marks the account as being closed at the end of the instruction’s execution, sending the rent exemption lamports to the specified |

Notice that we used a rather complex constraint to create an token account that has a PDA key (See this [code snippet](https://github.com/project-serum/anchor/blob/master/tests/misc/programs/misc/src/context.rs#L10) for more details). Let's take a closer look of it:

```rust=
#[derive(Accounts)]
#[instruction(token_bump: u8)]
pub struct TestTokenSeedsInit<'info> {
    #[account(
        init,
        seeds = [b"my-token-seed".as_ref()],
        bump = token_bump,
        payer = authority,
        token::mint = mint,
        token::authority = authority,
    )]
    pub my_pda: Account<'info, TokenAccount>,
    pub mint: Account<'info, Mint>,
    pub authority: AccountInfo<'info>,
    pub system_program: AccountInfo<'info>,
    pub rent: Sysvar<'info, Rent>,
    pub token_program: AccountInfo<'info>,
}

```

Check the [official document](https://docs.rs/anchor-lang/0.18.2/anchor_lang/derive.Accounts.html) for more constraints.

Now, the program should compile again successfully:

```bash
$ anchor build
...
```

## Build and Test

So far we have only accomplished the first part of the workflow. Let's write some client side test for it.

### Interface Description Language (IDL)

First, you can access the IDL via the following path:

```bash
$ cat ./target/idl/anchor_escrow.json
...
```

This will print the full IDL on the terminal:

```json=
// anchor_escrow.json

{
  "version": "0.0.0",
  "name": "anchor_escrow",
  "instructions": [
    {
      "name": "initialize",
      "accounts": [
        {
          "name": "initializer",
          "isMut": true,
          "isSigner": true
        },
        {
          "name": "mint",
          "isMut": false,
          "isSigner": false
        },
        {
          "name": "vaultAccount",
          "isMut": true,
          "isSigner": false
        },
        {
          "name": "initializerDepositTokenAccount",
          "isMut": true,
          "isSigner": false
        },
        {
          "name": "initializerReceiveTokenAccount",
          "isMut": false,
          "isSigner": false
        },
        {
          "name": "escrowAccount",
          "isMut": true,
          "isSigner": false
        },
        {
          "name": "systemProgram",
          "isMut": false,
          "isSigner": false
        },
        {
          "name": "rent",
          "isMut": false,
          "isSigner": false
        },
        {
          "name": "tokenProgram",
          "isMut": false,
          "isSigner": false
        }
      ],
      "args": [
        {
          "name": "vaultAccountBump",
          "type": "u8"
        },
        {
          "name": "initializerAmount",
          "type": "u64"
        },
        {
          "name": "takerAmount",
          "type": "u64"
        }
      ]
    },
    {
      "name": "cancel",
      "accounts": [
        {
          "name": "initializer",
          "isMut": true,
          "isSigner": true
        },
        {
          "name": "vaultAccount",
          "isMut": true,
          "isSigner": false
        },
        {
          "name": "vaultAuthority",
          "isMut": false,
          "isSigner": false
        },
        {
          "name": "initializerDepositTokenAccount",
          "isMut": true,
          "isSigner": false
        },
        {
          "name": "escrowAccount",
          "isMut": true,
          "isSigner": false
        },
        {
          "name": "tokenProgram",
          "isMut": false,
          "isSigner": false
        }
      ],
      "args": []
    },
    {
      "name": "exchange",
      "accounts": [
        {
          "name": "taker",
          "isMut": false,
          "isSigner": true
        },
        {
          "name": "takerDepositTokenAccount",
          "isMut": true,
          "isSigner": false
        },
        {
          "name": "takerReceiveTokenAccount",
          "isMut": true,
          "isSigner": false
        },
        {
          "name": "initializerDepositTokenAccount",
          "isMut": true,
          "isSigner": false
        },
        {
          "name": "initializerReceiveTokenAccount",
          "isMut": true,
          "isSigner": false
        },
        {
          "name": "initializer",
          "isMut": true,
          "isSigner": false
        },
        {
          "name": "escrowAccount",
          "isMut": true,
          "isSigner": false
        },
        {
          "name": "vaultAccount",
          "isMut": true,
          "isSigner": false
        },
        {
          "name": "vaultAuthority",
          "isMut": false,
          "isSigner": false
        },
        {
          "name": "tokenProgram",
          "isMut": false,
          "isSigner": false
        }
      ],
      "args": []
    }
  ],
  "accounts": [
    {
      "name": "EscrowAccount",
      "type": {
        "kind": "struct",
        "fields": [
          {
            "name": "initializerKey",
            "type": "publicKey"
          },
          {
            "name": "initializerDepositTokenAccount",
            "type": "publicKey"
          },
          {
            "name": "initializerReceiveTokenAccount",
            "type": "publicKey"
          },
          {
            "name": "initializerAmount",
            "type": "u64"
          },
          {
            "name": "takerAmount",
            "type": "u64"
          }
        ]
      }
    }
  ]
}
```

As you can see, the IDL basically defines everything needed for a client representation.

> You can think of IDL as **ABI** if you are familiar with Ethereum and Solidity.

Next, lets move to `tests/anchor-escrow.ts` to implement the tests.

### Setup

Before we dive into the actual test cases, let's first setup the boilerlate for the tests:

```bash
$ npm install --save @solana/spl-token
```

```typescript=
// anchor-escrow.ts

import * as anchor from '@project-serum/anchor';
import { Program } from '@project-serum/anchor';
import { AnchorEscrow } from '../target/types/anchor_escrow';
import { PublicKey, SystemProgram, Transaction } from '@solana/web3.js';
import { TOKEN_PROGRAM_ID, Token } from "@solana/spl-token";
import { assert } from "chai";

describe('anchor-escrow', () => {

  // Configure the client to use the local cluster.
  const provider = anchor.Provider.env();
  anchor.setProvider(provider);

  const program = anchor.workspace.AnchorEscrow as Program<AnchorEscrow>;

  let mintA = null;
  let mintB = null;
  let initializerTokenAccountA = null;
  let initializerTokenAccountB = null;
  let takerTokenAccountA = null;
  let takerTokenAccountB = null;
  let vault_account_pda = null;
  let vault_account_bump = null;
  let vault_authority_pda = null;

  const takerAmount = 1000;
  const initializerAmount = 500;

  const escrowAccount = anchor.web3.Keypair.generate();
  const payer = anchor.web3.Keypair.generate();
  const mintAuthority = anchor.web3.Keypair.generate();
  const initializerMainAccount = anchor.web3.Keypair.generate();
  const takerMainAccount = anchor.web3.Keypair.generate();

  it("Initialize program state", async () => {
    // TODO
  });

  it("Initialize escrow", async () => {
    // TODO
  });

  it("Exchange escrow state", async () => {
    // TODO
  });

  it("Initialize escrow and cancel escrow", async () => {
    // TODO
  });
});
```

> Note: `target/types/anchor_escrow` is generated by running `anchor build`. Make sure you build the program first.

You can see there are 4 test cases to be completed. However, the first test case `Initialize program state` is used for program state setup such as minting tokens. As a result, there should be only 3 test cases corresponding to 3 functions of the program.

Let's finish the program state initialization:

```typescript=
// anchor-escrow.ts

...

describe('anchor-escrow', () => {
  it("Initialize program state", async () => {
    // Airdropping tokens to a payer.
    await provider.connection.confirmTransaction(
      await provider.connection.requestAirdrop(payer.publicKey, 10000000000),
      "confirmed"
    );

    // Fund Main Accounts
    await provider.send(
      (() => {
        const tx = new Transaction();
        tx.add(
          SystemProgram.transfer({
            fromPubkey: payer.publicKey,
            toPubkey: initializerMainAccount.publicKey,
            lamports: 1000000000,
          }),
          SystemProgram.transfer({
            fromPubkey: payer.publicKey,
            toPubkey: takerMainAccount.publicKey,
            lamports: 1000000000,
          })
        );
        return tx;
      })(),
      [payer]
    );

    mintA = await Token.createMint(
      provider.connection,
      payer,
      mintAuthority.publicKey,
      null,
      0,
      TOKEN_PROGRAM_ID
    );

    mintB = await Token.createMint(
      provider.connection,
      payer,
      mintAuthority.publicKey,
      null,
      0,
      TOKEN_PROGRAM_ID
    );

    initializerTokenAccountA = await mintA.createAccount(initializerMainAccount.publicKey);
    takerTokenAccountA = await mintA.createAccount(takerMainAccount.publicKey);

    initializerTokenAccountB = await mintB.createAccount(initializerMainAccount.publicKey);
    takerTokenAccountB = await mintB.createAccount(takerMainAccount.publicKey);

    await mintA.mintTo(
      initializerTokenAccountA,
      mintAuthority.publicKey,
      [mintAuthority],
      initializerAmount
    );

    await mintB.mintTo(
      takerTokenAccountB,
      mintAuthority.publicKey,
      [mintAuthority],
      takerAmount
    );

    let _initializerTokenAccountA = await mintA.getAccountInfo(initializerTokenAccountA);
    let _takerTokenAccountB = await mintB.getAccountInfo(takerTokenAccountB);

    assert.ok(_initializerTokenAccountA.amount.toNumber() == initializerAmount);
    assert.ok(_takerTokenAccountB.amount.toNumber() == takerAmount);
  });
  ...

}
```

We should be able to pass the first test case at this point:

```bash
$ anchor test
...

  anchor-escrow
    ✔ Initialize program state (4814ms)
    ✔ Initialize escrow
    ✔ Exchange escrow state
    ✔ Initialize escrow and cancel escrow


  4 passing (5s)

✨  Done in 10.89s.
```

### Implement Tests for `initialize`, `exchange` and `cancel`

Next, we add the test case for `initialize`:

```typescript=
// anchor-escrow.ts

...

describe('anchor-escrow', () => {
  ...

  it("Initialize escrow", async () => {
    const [_vault_account_pda, _vault_account_bump] = await PublicKey.findProgramAddress(
      [Buffer.from(anchor.utils.bytes.utf8.encode("token-seed"))],
      program.programId
    );
    vault_account_pda = _vault_account_pda;
    vault_account_bump = _vault_account_bump;

    const [_vault_authority_pda, _vault_authority_bump] = await PublicKey.findProgramAddress(
      [Buffer.from(anchor.utils.bytes.utf8.encode("escrow"))],
      program.programId
    );
    vault_authority_pda = _vault_authority_pda;

    await program.rpc.initialize(
      vault_account_bump,
      new anchor.BN(initializerAmount),
      new anchor.BN(takerAmount),
      {
        accounts: {
          initializer: initializerMainAccount.publicKey,
          vaultAccount: vault_account_pda,
          mint: mintA.publicKey,
          initializerDepositTokenAccount: initializerTokenAccountA,
          initializerReceiveTokenAccount: initializerTokenAccountB,
          escrowAccount: escrowAccount.publicKey,
          systemProgram: anchor.web3.SystemProgram.programId,
          rent: anchor.web3.SYSVAR_RENT_PUBKEY,
          tokenProgram: TOKEN_PROGRAM_ID,
        },
        instructions: [
          await program.account.escrowAccount.createInstruction(escrowAccount),
        ],
        signers: [escrowAccount, initializerMainAccount],
      }
    );

    let _vault = await mintA.getAccountInfo(vault_account_pda);

    let _escrowAccount = await program.account.escrowAccount.fetch(
      escrowAccount.publicKey
    );

    // Check that the new owner is the PDA.
    assert.ok(_vault.owner.equals(vault_authority_pda));

    // Check that the values in the escrow account match what we expect.
    assert.ok(_escrowAccount.initializerKey.equals(initializerMainAccount.publicKey));
    assert.ok(_escrowAccount.initializerAmount.toNumber() == initializerAmount);
    assert.ok(_escrowAccount.takerAmount.toNumber() == takerAmount);
    assert.ok(
      _escrowAccount.initializerDepositTokenAccount.equals(initializerTokenAccountA)
    );
    assert.ok(
      _escrowAccount.initializerReceiveTokenAccount.equals(initializerTokenAccountB)
    );
  });
  ...

}
```

We should see 2 implemented test cases passed at this point:

```bash
$ anchor test
...

  anchor-escrow
    ✔ Initialize program state (5035ms)
    ✔ Initialize escrow (499ms)
    ✔ Exchange escrow state
    ✔ Initialize escrow and cancel escrow


  4 passing (6s)

✨  Done in 11.39s.
```

Similarly, let's implement the rest of the tests real quick:

```typescript=
// anchor-escrow.ts

...

describe('anchor-escrow', () => {
  ...

  it("Exchange escrow state", async () => {
    await program.rpc.exchange({
      accounts: {
        taker: takerMainAccount.publicKey,
        takerDepositTokenAccount: takerTokenAccountB,
        takerReceiveTokenAccount: takerTokenAccountA,
        initializerDepositTokenAccount: initializerTokenAccountA,
        initializerReceiveTokenAccount: initializerTokenAccountB,
        initializer: initializerMainAccount.publicKey,
        escrowAccount: escrowAccount.publicKey,
        vaultAccount: vault_account_pda,
        vaultAuthority: vault_authority_pda,
        tokenProgram: TOKEN_PROGRAM_ID,
      },
      signers: [takerMainAccount]
    });

    let _takerTokenAccountA = await mintA.getAccountInfo(takerTokenAccountA);
    let _takerTokenAccountB = await mintB.getAccountInfo(takerTokenAccountB);
    let _initializerTokenAccountA = await mintA.getAccountInfo(initializerTokenAccountA);
    let _initializerTokenAccountB = await mintB.getAccountInfo(initializerTokenAccountB);

    assert.ok(_takerTokenAccountA.amount.toNumber() == initializerAmount);
    assert.ok(_initializerTokenAccountA.amount.toNumber() == 0);
    assert.ok(_initializerTokenAccountB.amount.toNumber() == takerAmount);
    assert.ok(_takerTokenAccountB.amount.toNumber() == 0);
  });

  it("Initialize escrow and cancel escrow", async () => {
    // Put back tokens into initializer token A account.
    await mintA.mintTo(
      initializerTokenAccountA,
      mintAuthority.publicKey,
      [mintAuthority],
      initializerAmount
    );

    await program.rpc.initialize(
      vault_account_bump,
      new anchor.BN(initializerAmount),
      new anchor.BN(takerAmount),
      {
        accounts: {
          initializer: initializerMainAccount.publicKey,
          vaultAccount: vault_account_pda,
          mint: mintA.publicKey,
          initializerDepositTokenAccount: initializerTokenAccountA,
          initializerReceiveTokenAccount: initializerTokenAccountB,
          escrowAccount: escrowAccount.publicKey,
          systemProgram: anchor.web3.SystemProgram.programId,
          rent: anchor.web3.SYSVAR_RENT_PUBKEY,
          tokenProgram: TOKEN_PROGRAM_ID,
        },
        instructions: [
          await program.account.escrowAccount.createInstruction(escrowAccount),
        ],
        signers: [escrowAccount, initializerMainAccount],
      }
    );

    // Cancel the escrow.
    await program.rpc.cancel({
      accounts: {
        initializer: initializerMainAccount.publicKey,
        initializerDepositTokenAccount: initializerTokenAccountA,
        vaultAccount: vault_account_pda,
        vaultAuthority: vault_authority_pda,
        escrowAccount: escrowAccount.publicKey,
        tokenProgram: TOKEN_PROGRAM_ID,
      },
      signers: [initializerMainAccount]
    });

    // Check the final owner should be the provider public key.
    const _initializerTokenAccountA = await mintA.getAccountInfo(initializerTokenAccountA);
    assert.ok(_initializerTokenAccountA.owner.equals(initializerMainAccount.publicKey));

    // Check all the funds are still there.
    assert.ok(_initializerTokenAccountA.amount.toNumber() == initializerAmount);
  });
}
```

We should see all test cases passed at this moment:

```bash
$ anchor test
...

  anchor-escrow
    ✔ Initialize program state (4862ms)
    ✔ Initialize escrow (498ms)
    ✔ Exchange escrow state (503ms)
    ✔ Initialize escrow and cancel escrow (11043ms)


  4 passing (17s)

✨  Done in 22.90s.
```

And that's it!

## References

- https://github.com/ironaddicteddog/anchor-escrow
- https://hackmd.io/@ironaddicteddog/solana-starter-kit
- https://www.youtube.com/watch?v=cvW8EwGHw8U
- https://github.com/project-serum/anchor/blob/master/CHANGELOG.md
- https://paulx.dev/blog/2021/01/14/programming-on-solana-an-introduction/
- https://project-serum.github.io/anchor/getting-started/introduction.html
- https://docs.rs/anchor-lang/0.18.2/anchor_lang/derive.Accounts.html
- https://anchor.projectserum.com/
- https://blog.soteria.dev/?p=ef42d944f086

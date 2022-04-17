# A Starter Kit for Running Solana Validator

**Authors:** @eefylin, [@emersonliuuu](https://twitter.com/emersonliuuu), [@ironaddicteddog](https://twitter.com/ironaddicteddog)

***[Updated at 2022.3.31]***

## Overview

### Validator Landscape

![](https://hackmd.io/_uploads/SJe4wKbM9.png)

**See [Solana Beach](https://solanabeach.io/) for more details**
- 1600+ Validators
- 1400+ RPC Nodes
- 3000+ TPS
- < 1s Block Time
- ...

### Economics

Solana’s crypto-economic system is designed to promote a healthy, long term self-sustaining economy with participant incentives aligned to the security and decentralization of the network. The main participants in this economy are validation-clients who secure solana network.
At the early stage, just as many current blockchain economies (e.g. Bitcoin, Ethereum) does, rely on *protocol-based rewards* to support the economy, with the assumption that the revenue generated through *transaction fees* will support the economy in the long term, when the protocol derived rewards expire. see more [here](https://docs.solana.com/economics_overview)

**So, where do the protocol-based rewards come from?**
The answer is ***Inflation Rate*** and transaction fees.

Initial Inflation Rate: 8%
Dis-inflation Rate: −15%
Long-term Inflation Rate: 1.5%
![](https://hackmd.io/_uploads/HJViJCVGc.png)

**How can we avoid being affected by inflation?**
Stake your SOL and delegate to validator node.

## [Staking](https://solana.com/staking)

**Benefit**
1. Avoid token dilution acording to inflation of SOL
Staking tokens, which will receive their proportional distribution of inflation issuance, should assuage any dilution concerns for staked token holders.
2. Make Solana network more secure
As more token holders choose to stake their SOL tokens to different validators across the network, and the total amount of stake on the network increases, it becomes increasingly difficult for even a coordinated and well-funded attacker to amass enough stake to single-handedly alter the outcome of a consensus vote for their own benefit.

**Rewards**
people who stake their token earns their share by the formula and the figure below (or you can see [here](https://docs.solana.com/inflation/terminology#staking-yield-) for more detail). You might notice there's a negative relation between staking yield and total SOL staked, which may be a factor that influences stake/unstake behavior.
```
Staking Yield = Inflation Rate × Validator Uptime ×
                (1 − Validator Fee) × (1 / % SOL Staked)
where:
% SOL Staked = Total SOL Staked / Total Current Supply 
```

for example: (Statistics are from [here](https://staking.staked.us/solana-staking))

```
Inflation Rate:    4.3%
Validator Uptime: 99.5%
Validator Fee:    10.0%
% SOL Staked:     77.1%

Staking Yield = 0.043 x 0.99.5 x (1 - 0.1) x (1 / 0.771)
              = 0.0499 (4.99 %)
```

![](https://hackmd.io/_uploads/SkzVBeUz5.png)

**Risk (Slashing)**
"Slashing" is any process by which some portion of stake delegated to a validator is destroyed as a punitive measure for malicious actions undertaken by the validator. If you stake your stake to malicious validator, part of your stake portion might be slashed too.

malicious actions include inconsistant voting during lockout time, or nodes who cause some block fail to full finalization. See more slashing rules here[[1](https://docs.solana.com/proposals/slashing), [2](https://docs.solana.com/proposals/optimistic-confirmation-and-slashing)] 

**How to join**
1. [Stake and delegate to validator node](https://docs.solana.com/cli/delegate-stake)
2. [Join or create stake Pools](https://spl.solana.com/stake-pool)
3. Run a validator (talk more about this later)

## Validator

**Responsibility**
1. verified received block
2. sending [vote](https://docs.solana.com/terminology#ledger-vote) transaction (consensus mechanism)

**Minimum Requirement**
- minimum SOL:
0.02685864 SOL(vote account rent)
- hardware
CPU 12 cores / 24 threads, 
RAM 128GB, 
Disk 1.5 TB (Accounts: 500GB, Ledger: 1TB)
...
see more detail [here](https://docs.solana.com/running-validator/validator-reqs)

**Cost**

**1.1 SOL/day at most (vote transaction cost)**

**Rewards**
1. Protocol-based Rewards
Issuances from a global, protocol-defined, inflation rate(short term). These rewards are delivered on top of earnings from transaction fees (long term)
2. Transaction Fee
a fixed portion (initially 50%) of each transaction fee is destroyed, with the remaining fee going to the current leader processing the transaction.

## Terminology

![](https://hackmd.io/_uploads/ryruR6tM9.png)

- Validator vs RPC
RPC node is a validator node who provide full functionality for public to query on-chain data and send transaction and also improved reliability, which means it needs higher hardware requirement than general validator node.
- [vote and stake account](https://docs.solana.com/cluster/stake-delegation-and-rewards#vote-and-stake-accounts)
The rewards process is split into two on-chain programs. The Vote program solves the problem of making stakes slashable. The Stake program acts as custodian of the rewards pool and provides for passive delegation. The Stake program is responsible for paying rewards to staker and voter when shown that a staker's delegate has participated in validating the ledger. (Solana programs are stateless, thus we need accounts to store states)
- [identity](https://docs.solana.com/running-validator/validator-start#generate-identity)
same as keypair, in blockchain world *address* represent your identity.
- [paper wallet](https://docs.solana.com/wallet-guide/paper-wallet)
Solana commands can be run without ever saving a keypair to disk on a machine.
- [key rotation](https://docs.solana.com/running-validator/vote-accounts#key-rotation)
Leaders and validators are expected to use ephemeral keys for operation. And also for security concern, key rotation allows validator rotate the vote account authority keys with no effect on the stake accounts that have been delegate to the vote account.

## Before We Start

***Labs are timed and you cannot pause them**. The timer, which starts when you click Start Lab, shows how long Google Cloud resources will be made available to you.*

### Create a Quiklabs Account

*Here are the necessary steps to enroll in the test environment*

- Visit [https://ce.qwiklabs.com](https://ce.qwiklabs.com)
- Create your own account
- Verify your email by checking your email box
- Login to [https://ce.qwiklabs.com](https://ce.qwiklabs.com)
- **Fill our [meetup form](https://forms.gle/utvkviswqH8KNVNA8) with the email address you just used to create your account**

### Requirements

- Access to a standard internet browser (Chrome browser recommended).
- Time to complete the lab.

> If you already have your own personal Google Cloud account or project, do not use it for this lab.

> If you are using a Chrome OS device, open an Incognito window to run this lab.

## Setup

*This hands-on lab lets you do the lab activities yourself in a real cloud environment, not in a simulation or demo environment.*

***It does so by giving you new, temporary credentials that you use to sign in and access Google Cloud for the duration of the lab.***

### How to start your lab and sign in to the Google Cloud Console

1. Click the Start Lab button. If you need to pay for the lab, a pop-up opens for you to select your payment method. On the left is a panel populated with the temporary credentials that you must use for this lab.
2. Copy the username, and then click Open Google Console. The lab spins up resources, and then opens another tab that shows the Sign in page.
  - Open the tabs in separate windows, side-by-side.
  - If you see the Choose an account page, click Use Another Account. Choose an account
3. In the Sign in page, paste the username that you copied from the left panel. Then copy and paste the password.
> **Important: You must use the credentials from the left panel. Do not use your Google Cloud Training credentials. If you have your own Google Cloud account, do not use it for this lab (avoids incurring charges).**

4. Click through the subsequent pages:
  - Accept the terms and conditions.
  - Do not add recovery options or two-factor authentication (because this is a temporary account).
  - Do not sign up for free trials.
  - After a few moments, the Cloud Console opens in this tab.
  - Please check with the upper top selected the project assigned to you.

### Infrastructure Deployment

Create a VM (Navigation Menu -> Compute Engine -> VM instances -> Create Instance)
- Use the default name, region, zone.
- N2 series, Custom Machine type, CPU 24 cores, Ram 128 GB
- Change the "Boot Disk", "SSD persistent disk", Size 500GB
- Networking, Network tags, "solana"
- Leave the rest by default
- The above spec is based on [Solana document](https://docs.solana.com/running-validator/validator-reqs#hardware-recommendations)

Create 1 Firewall rule (Navigation Menu -> VPC network -> Firewall -> Create Firewall Rule)
- Name: solana-validator-ports
- Network: default
- Priority: 1000
- Diretion of traffic: Ingress
- Action on match: Allow
- Target tags: solana
- IPv4 ranges: 0.0.0.0/0
- tcp:8899, 8900, 11000
- udp:11000-11020

## Commands to start a Validator
### Connect to your Virtual Machine
- Please visit the VM page (Navigation Menu -> Compute Engine -> VM instances)
- Find the virtual machine you created, and click on the [SSH] buttom.
- Now let's run some scripts!

### Key and Configuration
Run following command to Solana Command Line Tool:

	sh -c "$(curl -sSfL https://release.solana.com/v1.9.13/install)"

Should get an output from previous command similiar to the following, please run it.

	export PATH="/home/<studnet-00-ID>/.local/share/solana/install/active_release/bin:$PATH"


By default, CLI connect to Mainnet Beta, Let's connect to Devnet

	solana config set --url http://api.devnet.solana.com


Leverage a System Tuner to update configuration automatically. For more detail please check [Solana document](https://docs.solana.com/running-validator/validator-start#system-tuning)

	sudo $(command -v solana-sys-tuner) --user $(whoami) > sys-tuner.log 2>&1 &

Solana CLI support 2 ways to create key pairs, file wallet or paper wallt. Let's use file wallet in this case for convinience. Paper wallet detail please check [here](https://docs.solana.com/wallet-guide/paper-wallet)

	solana-keygen new -o ~/.config/solana/validator-keypair.json
	
To show the pubkey again by running:

	solana-keygen pubkey ~/.config/solana/validator-keypair.json
	
Set the solana configuration to use your validator keypair for all following commands:

	solana config set --keypair ~/.config/solana/validator-keypair.json

You can use following command to check config status at any time:

	solana config get

To start your Validator, we need some SOL in the wallet. With Devnet we can simply get a SOL by following:

	solana airdrop 1

Let's check the balance

	solana balance

Before we launch our Validator, we also need another key-pair for Vote Account:

	solana-keygen new -o ~/.config/solana/vote-account-keypair.json

Also a key-pair for Authorized Withdrawer Account:

	solana-keygen new -o ~/.config/solana/authorized-withdrawer-keypair.json

Run this command to create your Vote Account:

	solana create-vote-account ~/.config/solana/vote-account-keypair.json ~/.config/solana/validator-keypair.json ~/.config/solana/authorized-withdrawer-keypair.json

### Create script for managed background running Validator
Create a executable file `validator.sh`

	touch ~/.config/solana/validator.sh

Use your prefered editor to pasta following content to `validator.sh`

	vim ~/.config/solana/validator.sh

```
solana-validator \
--identity ~/.config/solana/validator-keypair.json \
--vote-account ~/.config/solana/vote-account-keypair.json \
--rpc-port 8899 \
--entrypoint entrypoint.devnet.solana.com:8001 \
--limit-ledger-size \
--log ~/.config/solana/solana-validator.log \
--dynamic-port-range 11000-11020 &
```

Let's make it executable:

	chmod 755 ~/.config/solana/validator.sh

Now it's time to create a file for Systemd, which is the Linux program we are going to use"

	sudo touch /etc/systemd/system/sol.service

Paste following content to the file with your preferred editor: ( please change User, Environment, ExecStart with your own environment. )

	sudo vim /etc/systemd/system/sol.service

```
[Unit]
Description=Solana Validator
After=network.target
Wants=solana-sys-tuner.service
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=<student-00-ID>
LimitNOFILE=1000000
LogRateLimitIntervalSec=0
Environment="PATH=/bin:/usr/bin:/home/<student-00-ID>/.local/share/solana/install/active_release/bin"
ExecStart=/home/<student-00-ID>/.config/solana/validator.sh

[Install]
WantedBy=multi-user.target
```

Finally, let us run commands below to start our validator

	sudo systemctl daemon-reload #let systemd to load our new service

Enable the systemd, so when the service stop will bring up agaiin

	sudo systemctl enable --now sol #enable this service when VM restart

Now let's start the service

	bash ~/.config/solana/validator.sh

### Check your validator status and logs
Let's check the log, once you see your validator catch up with other validator, can move to next step.

	tail -f ~/.config/solana/solana-validator.log

Check our Validator from the Solana Devnet

	solana gossip | grep <PUBKEY>

If you see your pubkey and the IP matching your VM external IP, Your Done!

## [Delegation Program](https://solana.foundation/delegation-program)
**Goal**
Incentivize new validators to join to secure Solana network

**Get delegation from fundation**
- meet the Testnet Participation Criteria and all of the Baseline Criteria
-> receive a “baseline” delegation from the Solana Foundation of 25,000 SOL

[Example: Baseline criteria for Epoch 252](https://solana.foundation/delegation-criteria/#vote-credits)
| BASELINE REQUIREMENT         | RESULT                                  |
| ---------------------------- | --------------------------------------- |
| Vote Credits                 | 227,047 or more                         |
| Maximum Commission           | 10% or under                            |
| Solana Release               | 1.7.14 or greater                       |
| Self Stake                   | 100 or more                             |
| Total Stake                  | 3,000,000 or less                       |
| Infrastructure Concentration | 10% or less                             |
| Infrastructure Concentration | Baseline in 5 of last 10 testnet epochs |

- meets all criteria to receive the baseline delegation and also meets all of the Bonus Criteria
-> receive a “bonus” delegation (size is dynamic, this part depends on current participants)


**Get delegation from external**
1. starting a website and explaining why delegators should stake to you
2. starting a stake pool that promotes decentralization
3. joining a stake pool (https://solana.foundation/stake-pools) and receiving additional delegations from them.
 
([source](https://discord.com/channels/428295358100013066/849749936916267029/954445282572111902))

---

## References

- https://github.com/DappioWonderland/solana
- https://docs.solana.com/running-validator/validator-start
- https://hackmd.io/@ironaddicteddog/solana-starter-kit
- https://github.com/DappioWonderland/solana

###### tags: `notes`

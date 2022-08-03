# stakewars-iii-instruction
A guide to completing NEAR’s Stake War III challenges

Official Manual: [https://github.com/near/stakewars-iii/tree/main/challenges](https://github.com/near/stakewars-iii/tree/main/challenges)

# **Challenge 001**

### 001 - **Create a wallet on shardnet**

Follow instructions on [https://wallet.shardnet.near.org/](https://wallet.shardnet.near.org/)

### 001 - **Setup Near CLI**

```bash
# Update apt
sudo apt update && sudo apt upgrade -y

# Install Node.js and npm
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -  
sudo apt install build-essential nodejs
PATH="$PATH"

# Check version
node -v
v18.6.0

npm -v
8.13.2

# Install near-cli
sudo npm install -g near-cli

# For this chunk-only producer, we'll be using shardnet
echo 'export NEAR_ENV=shardnet' >> ~/.profile && source .profile
```

**CLI commands**

*Proposals*

A proposal by a validator indicates they would like to enter the validator set, in order for a proposal to be accepted it must meet the minimum seat price.

Command: `near proposals`

![Screen Shot 2022-07-14 at 9.41.25 AM.png](https://github.com/dlpigpen/stakewars-iii-instruction/blob/main/images/Screen%20Shot%202022-07-19%20at%2008.12.34.png?raw=true)

*Validators Current*

This shows a list of active validators in the current epoch, the number of blocks produced, number of blocks expected, and online rate. Used to monitor if a validator is having issues.

Command: `near validators current`

![Screen Shot 2022-07-14 at 9.41.41 AM.png](https://github.com/dlpigpen/stakewars-iii-instruction/blob/main/images/Screen%20Shot%202022-07-19%20at%2008.11.34.png?raw=true)

*Validators Next*

This shows validators whose proposal was accepted one epoch ago, and that will enter the validator set in the next epoch.

Command: `near validators next`

![Screen Shot 2022-07-14 at 9.41.49 AM.png](https://github.com/dlpigpen/stakewars-iii-instruction/blob/main/images/Screen%20Shot%202022-07-19%20at%2008.13.00.png?raw=true)

Full list of Near CLI commands: [https://github.com/near/near-cli](https://github.com/near/near-cli)

---

# **Challenge 002**

### 002 - **Setup your validator**

For chunk-only producers, the hardware requirements are reduced significantly. Recommended specs are as following,

**Dependencies**

```bash
# Check system compatibility
lscpu | grep -P '(?=.*avx )(?=.*sse4.2 )(?=.*cx16 )(?=.*popcnt )' > /dev/null \
  && echo "Supported" \
  || echo "Not supported"
Supported

# Dependencies
sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python3 docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo

# Install Python pip
sudo apt install python3-pip

# Include base bin path
USER_BASE_BIN=$(python3 -m site --user-base)/bin
export PATH="$USER_BASE_BIN:$PATH"

# Install build tools
sudo apt install clang build-essential make

# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

**Build nearcore binary**

```bash
# Clone nearcore repo
git clone https://github.com/near/nearcore
cd nearcore
git fetch origin --tags
git checkout 68bfa84ed1455f891032434d37ccad696e91e4f5

#Updated on July 22 new build:
git fetch
git checkout 0f81dca95a55f975b6e54fe6f311a71792e21698
cargo build -p neard --release --features shardnet
sudo systemctl restart neard 

# Build the binary, this will take some time
cargo build -p neard --release --features shardnet

# Check version
./target/release/neard -V
neard (release 1.28.0-rc.2) (build crates-0.14.0-145-g86b95c353-modified) (rustc 1.61.0) (protocol 100) (db 31)
```

**Initialize data directories**

```bash
./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis

# Replace config.json
rm ~/.near/config.json
wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json

# Download data snapshot
# This step has been removed for the latest commit: git checkout 8448ad1ebf27731a43397686103aa5277e7f2fcf
sudo apt-get install awscli -y
# cd ~/.near
# aws s3 --no-sign-request cp s3://build.openshards.io/stakewars/shardnet/data.tar.gz .  
# tar -xzvf data.tar.gz

# Start the node
cd nearcore
./target/release/neard --home ~/.near run
```

We will configure a system service for the daemon in later steps, for now keep the process running,  download all headers and sync to the latest block height. (This will take a fair amount of time).

![Screen Shot 2022-07-14 at 1.42.16 PM.png](https://github.com/dlpigpen/stakewars-iii-instruction/blob/main/images/Screen_Shot_2022-07-14_at_1.42.16_PM%20(1).png?raw=true)

### 002 - Configure keys

Go to your Near-CLI instance from challenge 001

For better clarity, we’ll provide examples for these parameters:

<account_id>: mitsorilab2.shardnet.near

<pool_name>: mitsorilab2

<pool_id>: mitsorilab2.factory.shardnet.near

**Install access key locally**

```bash
near login
```

Copy the link and launch it your browser.

![Screen Shot 2022-07-14 at 2.18.07 PM.png](https://github.com/dlpigpen/stakewars-iii-instruction/blob/main/images/Screen%20Shot%202022-07-19%20at%2008.34.36.png?raw=true)

Your account ID is the one created from [001 - **Create a wallet on shardnet**](https://www.notion.so/001-Create-a-wallet-on-shardnet-86037592975c4999ab998d99936269a2). The page should go to unreachable state after you confirm.

![Screen Shot 2022-07-14 at 2.18.17 PM.png](https://github.com/dlpigpen/stakewars-iii-instruction/blob/main/images/Screen_Shot_2022-07-14_at_2.18.17_PM.png?raw=true)

Go back to the terminal and enter the same ID to complete login.

![Screen Shot 2022-07-14 at 2.19.00 PM.png](https://github.com/dlpigpen/stakewars-iii-instruction/blob/main/images/Screen%20Shot%202022-07-19%20at%2008.36.53.png?raw=true)

**Make a validator_key from your wallet**

```bash
cp ~/.near-credentials/shardnet/mitsorilab2.shardnet.near.json ~/.near/validator_key.json
```

**Edit validator_key.json**

```bash
nano $HOME/.near/validator_key.json

{
	"account_id":"mitsorilab2.shardnet.near",
	"public_key":"ed25519:GEq98E9ZM5VKHvmRMoHnamD5xXspAGprUGQFQF7vuu7E",
	"private_key":"ed25519:*******************************************"
}
```

Edit `account_id` to <pool_name>.factory.shardnet.near and `private_key` param to `secret_key`

You should end up with something like this

```bash
{
	"account_id":"mitsorilab2.factory.shardnet.near",
	"public_key":"ed25519:GEq98E9ZM5VKHvmRMoHnamD5xXspAGprUGQFQF7vuu7E",
	"secret_key":"ed25519:*******************************************"
}
```

**Create system service**

```bash
sudo nano /etc/systemd/system/neard.service

[Unit]
Description=neard
After=network-online.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/.near
ExecStart=/home/ubuntu/nearcore/target/release/neard run   
# Optionally you can output the logs to files
#StandardOutput=file:/home/ubuntu/.near/neard.log
#StandardError=file:/home/ubuntu/.near/nearderror.log
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=45
KillMode=mixed

[Install]
WantedBy=multi-user.target

# Enable daemon and start service
sudo sytemctl enable neard.service
sudo systemctl start neard

# For pretty log printing
sudo apt install ccze

# Monitor log
journalctl -fu neard | ccze -A
```

![Screen Shot 2022-07-14 at 5.58.47 PM.png](https://github.com/dlpigpen/stakewars-iii-instruction/blob/main/images/Screen%20Shot%202022-07-19%20at%2008.37.19.png?raw=true)

---

# **Challenge 003**

<account_id>: mitsorilab2.shardnet.near

<pool_name>: mitsorilab2

<pool_id>: mitsorilab2.factory.shardnet.near

### 003 - Deploy a staking pool contract

**Deploy a staking pool**

```bash
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "<pool name>", "owner_id": "<account_id>", "stake_public_key": "<public key>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="<account_id>" --amount=30 --gas=300000000000000
```

**Example**

```bash
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "mitsorilab2", "owner_id": "mitsorilab2.shardnet.near", "stake_public_key": "GEq98E9ZM5VKHvmRMoHnamD5xXspAGprUGQFQF7vuu7E", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="mitsorilab2.shardnet.near" --amount=30 --gas=300000000000000
```

Upon completion, you should receive an URL similar to this
[https://explorer.shardnet.near.org/transactions/8rFah6Uus4y6jUhbGCg6gxH8YDTfvGv7ZkjJrcYWgijD](https://explorer.shardnet.near.org/transactions/8rFah6Uus4y6jUhbGCg6gxH8YDTfvGv7ZkjJrcYWgijD). Visit the link in your browser to confirm the transaction on the tracker.

![Screen Shot 2022-07-14 at 8.51.39 PM.png](https://github.com/dlpigpen/stakewars-iii-instruction/blob/main/images/Screen%20Shot%202022-07-19%20at%2008.39.10.png?raw=true)

**Staking to the pool**

```bash
near call <pool_id> deposit_and_stake --amount <NEAR amount> --accountId <account_id> --gas=300000000000000
```

**Example**

```bash
near call mitsorilab2.factory.shardnet.near deposit_and_stake --amount 550 --accountId mitsorilab2.shardnet.near --gas=300000000000000
```

**View staked balance**

```bash
near view mitsorilab2.factory.shardnet.near get_account_staked_balance '{"account_id": "mitsorilab2.shardnet.near"}'
```

**Misc functions you can invoke**

```bash
# Unstake yoctoNEAR
near call <pool_id> unstake '{"amount": "<amount yoctoNEAR>"}' --accountId <account_id> --gas=300000000000000

near call mitsorilab2.factory.shardnet.near unstake '{"amount": "1501013553987867946208039512"}' --accountId mitsorilab2.shardnet.near --gas=300000000000000

# Unstake all NEAR
near call <pool_id> unstake_all --accountId <account_id> --gas=300000000000000

# Withdraw yoctoNEAR
near call <pool_id> withdraw '{"amount": "<amount yoctoNEAR>"}' --accountId <account_id> --gas=300000000000000

# Withdraw all NEAR
near call <pool_id> withdraw_all --accountId <account_id> --gas=300000000000000

# Staked Balance
near view <pool_id> get_account_staked_balance '{"account_id": "<account_id>"}'

# Unstaked Balance
near view <pool_id> get_account_unstaked_balance '{"account_id": "<account_id>"}'

near view mitsorilab2.factory.shardnet.near get_account_unstaked_balance '{"account_id": "mitsorilab2.shardnet.near"}'

# Available for Withdrawal
near view <pool_id> is_account_unstaked_balance_available '{"account_id": "<account_id>"}'

near view mitsorilab2.factory.shardnet.near is_account_unstaked_balance_available '{"account_id": "mitsorilab2.shardnet.near"}'

# Pause Staking
near call <pool_id> pause_staking '{}' --accountId <account_id>

# Resume Staking
near call <pool_id> resume_staking '{}' --accountId <account_id>
```

**Submit the proposal**

In order to have a validator seat, you must submit a proposal with a ping. A ping issues a new proposal and updates the staking balances for your delegators. A ping should be issued each epoch to keep reported rewards current.

```bash
near call <pool_id> ping '{}' --accountId <account_id> --gas=300000000000000
```

**Example**

```bash
near call mitsorilab2.factory.shardnet.near ping '{}' --accountId mitsorilab2.shardnet.near --gas=300000000000000
```

Visit this link: [https://explorer.shardnet.near.org/nodes/validators](https://explorer.shardnet.near.org/nodes/validators) to see if the proposal has been submitted properly, you should see something like this

![Screen Shot 2022-07-14 at 9.29.32 PM.png](https://github.com/dlpigpen/stakewars-iii-instruction/blob/main/images/Screen%20Shot%202022-07-19%20at%2008.39.46.png?raw=true)

**Ping**

```bash
near call <pool_id> ping '{}' --accountId <account_id> --gas=300000000000000
```

**Example**

```bash
near call mitsorilab2.factory.shardnet.near ping '{}' --accountId mitsorilab2.shardnet.near --gas=300000000000000
```

A ping issues a new proposal and updates the staking balances for your delegators. A ping should be issued each epoch to keep reported rewards current.

**Create a cron job to run ping every epoch**

```bash
crontab -e 

# This needs to be set again else it'll look for testnet by default
NEAR_ENV=shardnet
# Current epoch is around ~2.5h, we'll invoke ping every hour for now
0 * * * * near call mitsorilab2.factory.shardnet.near ping '{}' --accountId mitsorilab2.shardnet.near --gas=300000000000000 >> $HOME/cron.log
```

### 003 - Becoming a validator

- [x]  The node must be fully synced
- [x]  The `validator_key.json` must be in place
- [x]  The contract must be initialized with the public_key in `validator_key.json`
- [x]  The account_id must be set to the staking pool contract id
- [x]  There must be enough delegations to meet the minimum seat price. See the seat price [here](https://explorer.shardnet.near.org/nodes/validators).
- [x]  A proposal must be submitted by pinging the contract
- [x]  Once a proposal is accepted a validator must wait 2-3 epoch to enter the validator set
- [x]  Once in the validator set the validator must produce great than 90% of assigned blocks

Visit [https://explorer.shardnet.near.org/nodes/validators](https://explorer.shardnet.near.org/nodes/validators) and make sure your validator stays active.

![Screen Shot 2022-07-15 at 7.13.01 PM.png](https://github.com/dlpigpen/stakewars-iii-instruction/blob/main/images/Screen%20Shot%202022-07-19%20at%2008.45.23.png?raw=true)

---

# **Challenge 004**

### 004 - ****Monitor and make alerts****

Our node is doing fine, but we need to check status periodically, because we can’t see terminal every time, we need some thing that will inform us if something will happen with our node.

I made a small script for notifications of changes in online status, validator status and peers. It will help to react quickly and help keep high uptime and avoid slashing. I will run this script every 60 sec. using cron.

Script will send notifications to my telegram if something changes with node.

Create a telegram bot using using the guide in Telegram docs: https://core.telegram.org/bots#creating-a-new-bot

BotFather tell your new telegram_bot_token
To get telegram_chat_id you can use: @getmyid_bot

Get My ID bot sent your user id
Now we have telegram_bot_token and telegram_chat_id
```
cd ~
git clone https://github.com/Klesh-/near-protocol-node-telegram-notifications.git
cd near-protocol-node-telegram-notifications
```
Clone .env.example into .env
```
cp .env.example .env
```
- Edit .env file and fill parameters:
- TG_API_KEY to bot token which you got from BotFather
- TG_CHAT_ID to user id which you got from getmyid_bot
- POOL_ID to {PoolName}.factory.shardnet.near for me it’s klesh.factory.shardnet.near
- Give execution permissions:
```
chmod +x ./report_node_status.sh
```
Test script:
Open dialog with @<YourBotName> and click /start. Now your own bot allowed to send you messages.
Run script
```
./report_node_status.sh
```
Message should appear in telegram from bot that you created

Bot respond to us with latest status changes in node
Now we need to let this script to be called every 1 min.
```
crontab -e
```
Put command as a cron job for every minute. Replace UserDirectory with path to user directory.
```
* * * * * /<UserDirectory>/near-protocol-node-telegram-notifications/report_node_status.sh &> /dev/null
```

Crontab job to check node status using script
Script using node RPC and NEAR-CLI to get status.
It watching: prev_epoch_kickout, current_validators, next_validators, current_proposals from CLI command response to get status of validator.
Here is and example of notifications

Telegram notifications
Setup Grafana to get detailed information about our node.
I made easy to setup repository with Grafana + Prometheus and preloaded dashboard with alerts on all important parameters of node.
All you need is clone, configure and run.
GitHub - Klesh-/near-protocol-node-monitoring: A monitoring tool based on Grafana + Prometheus for…
A monitoring tool based on Grafana + Prometheus for Near Protocol Node - GitHub - Klesh-/near-protocol-node-monitoring…
github.com

Clone repository into home directory
```
cd ~
git clone https://github.com/Klesh-/near-protocol-node-monitoring.git
cd near-protocol-node-monitoring
chmod +x start.sh
```
Clone .env.example into .env
```
cp .env.example .env
```
Edit .env file and fill parameters:
GRAFANA_ADMIN_PASSWORD to admin password of Grafana
Run monitoring service
```
./start.sh
```
After it download all necessary docker images you can open http://{YourNodeIP}:19000 and see Grafana welcome screen.
Use login admin and password that you set earlier.
Navigate to Alerting > Notification channels > Add channel
Add telegram channel using bot token and chat id from previous step.

Save and navigate to Near Node dashboard.

Now you have super detailed information about what happening in Near node and will get alerts to telegram if any of these parameters are out of range.

# **Challenge 005**

### 005 - Mount a node validator on AWS

This guide was already created on AWS, as seen in challenge 002. We’ll document final specs and pricing in this challenge.

**Operating System**

Ubuntu Server 22.04 LTS (HVM), SSD Volume Type

**Recommended (tg4.2xlarge)**

**Our spec (tg4.2xlarge)**

We find the recommended specs to be sufficient for chunk production, but still upgraded it due to that fact that we’re running a few other services, including monitoring and alerting on the same instance. 

### 005 - Pricing

To create a pricing estimate, visit the [AWS calculator](https://calculator.aws/#/estimate)

tg4.2xlarge with 200gb SSD will run for ~$3 per month on a one year EC2 Instance All Upfront Plans

![Screen Shot 2022-07-16 at 4.40.07 PM.png](https://github.com/dlpigpen/stakewars-iii-instruction/blob/main/images/Screen%20Shot%202022-07-20%20at%2011.40.52.png?raw=true)

---

# **Challenge 006**

### 006 - **Ping the network**

**Create directories to store logs**

```bash
cd $HOME && mkdir scripts && mkdir logs
cd scripts
nano ping.sh
```

<user_id> is retrieved in the terminal with `echo $USER`

<account_id>: mitsorilab2.shardnet.near

<pool_id>: mitsorilab2.factory.shardnet.near

**Create a ping script**

Enter the following to `ping.sh`

```bash
#!/bin/sh
# Ping call to renew Proposal added to crontab

export NEAR_ENV=shardnet
export LOGS=/root/<user_id>/logs
export POOLID=<pool_id>
export ACCOUNTID=<account_id>

echo "---" >> $LOGS/all.log
date >> $LOGS/all.log
near call $POOLID ping '{}' --accountId $ACCOUNTID --gas=300000000000000 >> $LOGS/all.log
near proposals | grep $POOLID >> $LOGS/all.log
near validators current | grep $POOLID >> $LOGS/all.log
near validators next | grep $POOLID >> $LOGS/all.log
```

**Update crontab**

```bash
crontab -e

# Fill this in
*/5 * * * * sh /home/<user_id>/scripts/ping.sh
```

Check under `/home/<user_id>/logs/all.log` for ping history
	
## Docker
Here is the the script for who wants to use Docker file:
```
FROM ubuntu:latest as build
USER root
ENV NEAR_ENV=shardnet
ENV DEBIAN_FRONTEND noninteractive
WORKDIR /tools/rust
RUN apt update && apt dist-upgrade -y && apt install -y mc wget telnet git curl iotop atop vim && apt-get clean autoclean clang build-essential make libclang-dev && apt-get autoremove --yes && rm -rf /var/lib/{apt,dpkg,cache,log}/
RUN curl https://sh.rustup.rs -sSf | bash -s -- -y
WORKDIR /tools
RUN apt update && apt dist-upgrade -y && apt install -y build-essential libclang-dev && apt-get autoremove --yes && rm -rf /var/lib/{apt,dpkg,cache,log}/
RUN echo 2
RUN git clone https://github.com/near/nearcore /tools/nearcore
WORKDIR /tools/nearcore
#RUN git checkout 1.28.0
RUN git checkout 0f81dca95a55f975b6e54fe6f311a71792e21698
RUN ls /root/.cargo/bin
ENV PATH="/root/.cargo/bin:$PATH" 
RUN cargo build -p neard --release --features shardnet
ENV PATH="/tools/nearcore/target/release:$PATH" 

FROM ubuntu:latest
USER root
ENV DEBIAN_FRONTEND noninteractive
RUN apt update && apt dist-upgrade -y && apt install -y mc wget telnet git curl iotop atop vim jq && apt-get autoremove --yes && rm -rf /var/lib/{apt,dpkg,cache,log}/
RUN useradd -d /app -ms /bin/bash app
COPY start.sh /app/start.sh
RUN chown app:app /app/start.sh && chmod 0700 /app/start.sh
USER app
WORKDIR /app
COPY --from=build /tools/nearcore/target/release/neard /app/near/neard
ENV PATH="/app/near:$PATH" 
RUN echo 'export NEAR_ENV=shardnet' >> /app/.bashrc
ENTRYPOINT [ "/app/start.sh" ]
```

# **Challenge 008**

| Task | Notes |
| --- | --- |
| 008 - Deploy a reward split contract |  |

### 008 - Deploy a reward split contract

The idea of this smart contract is to deploy to an account that owns a staking pool. With the smart contract deployed, the owner of the staking pool will be able redistribute validator rewards to other accounts.

**Dependencies**

```bash
# Install rust
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
source $HOME/.cargo/env

# Add wasm toolchain
rustup target add wasm32-unknown-unknown
```

**Clone project repo**

```bash
git clone https://github.com/zavodil/near-staking-pool-owner
```

**Compile the smart contract**

```bash
cd near-staking-pool-owner/contract
cargo build --target wasm32-unknown-unknown --release or
sh build.sh
```

**Deploy the contract**

```bash
NEAR_ENV=shardnet near deploy <OWNER_ID>.shardnet.near --wasmFile target/wasm32-unknown-unknown/release/contract.wasm
```

Example

```bash
NEAR_ENV=shardnet near deploy mdstaking.shardnet.near --wasmFile target/wasm32-unknown-unknown/release/contract.wasm
```

Visit the explorer and make sure the contract is successfully deployed: [https://explorer.shardnet.near.org/transactions/2iEpqzYAN76o6aBzmSGVVx4vykctB7YrYjsWy1Q6sbt](https://explorer.shardnet.near.org/transactions/2iEpqzYAN76o6aBzmSGVVx4vykctB7YrYjsWy1Q6sbtY)

![Screen Shot 2022-07-27 at 12.08.40 PM.png](https://github.com/dlpigpen/stakewars-iii-instruction/blob/main/images/Screen%20Shot%202022-07-27%20at%2023.37.34.png?raw=true)

**Split to two other accounts**

First create two more wallets on [https://wallet.shardnet.near.org/](https://wallet.shardnet.near.org/)

In our case, our two wallets are `mdstakingu1.shardnet.near` and `mdstakingu2.shardnet.near`

Env variable

```bash
export CONTRACT_ID=mdstaking.shardnet.near
```

Initialize the smart contract with these accounts

```bash
near call $CONTRACT_ID new '{"staking_pool_account_id": "<STAKINGPOOL_ID>.factory.shardnet.near", "owner_id":"<OWNER_ID>.shardnet.near", "reward_receivers": [["<SPLITED_ACCOUNT_ID_1>.shardnet.near", {"numerator": 3, "denominator":10}], ["<SPLITED_ACCOUNT_ID_2>.shardnet.near", {"numerator": 70, "denominator":100}]]}' --accountId $CONTRACT_ID
```

Example

```bash
# This will send 30% of pool rewards to ll1 and 70% to ll2

near call $CONTRACT_ID new '{"staking_pool_account_id": "mitsorilab2.factory.shardnet.near", "owner_id":"mitsorilab.shardnet.near", "reward_receivers": [["mdstakingu1.shardnet.near", {"numerator": 3, "denominator":10}], ["mdstakingu2.shardnet.near", {"numerator": 70, "denominator":100}]]}' --accountId $CONTRACT_ID
```

**Withdraw rewards**

```bash
# Call the withdraw method on the owner account, this action will unstake & withdraw service fee received by pool and distribute it among the reward receivers.

NEAR_ENV=shardnet near call $CONTRACT_ID withdraw '{}' --accountId $CONTRACT_ID --gas 200000000000000
```

Example

```bash
NEAR_ENV=shardnet near call $CONTRACT_ID withdraw '{}' --accountId $CONTRACT_ID --gas 200000000000000
```

Link: https://explorer.shardnet.near.org/transactions/CMhsP2fegkYHtGwBuvhNACAJJyg9v4PKTuoWzKuQaLcD
	
![Screen Shot 2022-07-27 at 12.53.12 PM.png](https://github.com/dlpigpen/stakewars-iii-instruction/blob/main/images/Screen%20Shot%202022-07-27%20at%2023.46.37.png?raw=true)

Note, this withdraw invocation not only withdraws the validator rewards, but also unstakes all self-delegations. The current unstaking period is 4-6 epochs, so you can either wait to unlock, or alternatively, you could just send some more balances to the owner account and stake again. Just make sure to stay above the seat price.

Example

```bash
near call mitsorilab2.factory.shardnet.near deposit_and_stake --amount 450 --accountId mitsorilab.shardnet.near --gas=300000000000000
```
	
## Mitsori Labs

Website: [https://hamado-ltd.com/](https://hamado-ltd.com)

Twitter: [https://twitter.com/mitsori_md](https://twitter.com/mitsori_md)

Github: https://github.com/dlpigpen](https://github.com/dlpigpen)

Chunk-Only Producer: [mitsorilab2.factory.shardnet.near](https://explorer.shardnet.near.org/accounts/mitsorilab.factory.shardnet.near)

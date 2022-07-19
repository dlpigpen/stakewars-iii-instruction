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

![Screen Shot 2022-07-14 at 9.41.25 AM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2e112eca-e2fb-47c0-995b-1912a6e9fcda/Screen_Shot_2022-07-14_at_9.41.25_AM.png)

*Validators Current*

This shows a list of active validators in the current epoch, the number of blocks produced, number of blocks expected, and online rate. Used to monitor if a validator is having issues.

Command: `near validators current`

![Screen Shot 2022-07-14 at 9.41.41 AM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a6331c2e-feac-4820-a1a3-e29f2f7fa0b5/Screen_Shot_2022-07-14_at_9.41.41_AM.png)

*Validators Next*

This shows validators whose proposal was accepted one epoch ago, and that will enter the validator set in the next epoch.

Command: `near validators next`

![Screen Shot 2022-07-14 at 9.41.49 AM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/266ec70c-8945-445b-823f-4dc626f81ac7/Screen_Shot_2022-07-14_at_9.41.49_AM.png)

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
git checkout 1.28.0-rc.2

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
sudo apt-get install awscli -y
cd ~/.near
aws s3 --no-sign-request cp s3://build.openshards.io/stakewars/shardnet/data.tar.gz .  
tar -xzvf data.tar.gz

# Start the node
cd nearcore
./target/release/neard --home ~/.near run
```

We will configure a system service for the daemon in later steps, for now keep the process running,  download all headers and sync to the latest block height. (This will take a fair amount of time).

![Screen Shot 2022-07-14 at 1.42.16 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1ba2dcfc-eba8-4533-8740-dd46b5767697/Screen_Shot_2022-07-14_at_1.42.16_PM.png)

### 002 - Configure keys

Go to your Near-CLI instance from challenge 001

For better clarity, we’ll provide examples for these parameters:

<account_id>: lydialabs.shardnet.near

<pool_name>: lydialabs

<pool_id>: lydialabs.factory.shardnet.near

**Install access key locally**

```bash
near login
```

Copy the link and launch it your browser.

![Screen Shot 2022-07-14 at 2.18.07 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/12e9d735-5235-46e6-a155-7b362cf71022/Screen_Shot_2022-07-14_at_2.18.07_PM.png)

Your account ID is the one created from [001 - **Create a wallet on shardnet**](https://www.notion.so/001-Create-a-wallet-on-shardnet-86037592975c4999ab998d99936269a2). The page should go to unreachable state after you confirm.

![Screen Shot 2022-07-14 at 2.18.17 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f2231628-9406-4b31-a1ee-b868d75b8478/Screen_Shot_2022-07-14_at_2.18.17_PM.png)

Go back to the terminal and enter the same ID to complete login.

![Screen Shot 2022-07-14 at 2.19.00 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c2700f96-d1ee-4b99-89f3-8ebf30c243f8/Screen_Shot_2022-07-14_at_2.19.00_PM.png)

**Make a validator_key from your wallet**

```bash
cp ~/.near-credentials/shardnet/lydialabs.shardnet.near.json ~/.near/validator_key.json
```

**Edit validator_key.json**

```bash
nano $HOME/.near/validator_key.json

{
	"account_id":"lydialabs.shardnet.near",
	"public_key":"ed25519:GEq98E9ZM5VKHvmRMoHnamD5xXspAGprUGQFQF7vuu7E",
	"private_key":"ed25519:*******************************************"
}
```

Edit `account_id` to <pool_name>.factory.shardnet.near and `private_key` param to `secret_key`

You should end up with something like this

```bash
{
	"account_id":"lydialabs.factory.shardnet.near",
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

![Screen Shot 2022-07-14 at 5.58.47 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/69c262e2-4d17-4bbb-8dd5-867884a85727/Screen_Shot_2022-07-14_at_5.58.47_PM.png)

---

# **Challenge 003**

<account_id>: lydialabs.shardnet.near

<pool_name>: lydialabs

<pool_id>: lydialabs.factory.shardnet.near

### 003 - Deploy a staking pool contract

**Deploy a staking pool**

```bash
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "<pool name>", "owner_id": "<account_id>", "stake_public_key": "<public key>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="<account_id>" --amount=30 --gas=300000000000000
```

**Example**

```bash
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "lydialabs", "owner_id": "lydialabs.shardnet.near", "stake_public_key": "GEq98E9ZM5VKHvmRMoHnamD5xXspAGprUGQFQF7vuu7E", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="lydialabs.shardnet.near" --amount=30 --gas=300000000000000
```

Upon completion, you should receive an URL similar to this
[https://explorer.shardnet.near.org/transactions/3R96WYapQvc5CmJJ6GLZRP2NWv7S4a9nFVipJ2Y7V1zv](https://explorer.shardnet.near.org/transactions/3R96WYapQvc5CmJJ6GLZRP2NWv7S4a9nFVipJ2Y7V1zv). Visit the link in your browser to confirm the transaction on the tracker.

![Screen Shot 2022-07-14 at 8.51.39 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3796dd40-45cb-4703-9a73-727322f8bc24/Screen_Shot_2022-07-14_at_8.51.39_PM.png)

**Staking to the pool**

```bash
near call <pool_id> deposit_and_stake --amount <NEAR amount> --accountId <account_id> --gas=300000000000000
```

**Example**

```bash
near call lydialabs.factory.shardnet.near deposit_and_stake --amount 550 --accountId lydialabs.shardnet.near --gas=300000000000000
```

**View staked balance**

```bash
near view lydialabs.factory.shardnet.near get_account_staked_balance '{"account_id": "lydialabs.shardnet.near"}'
```

**Misc functions you can invoke**

```bash
# Unstake yoctoNEAR
near call <pool_id> unstake '{"amount": "<amount yoctoNEAR>"}' --accountId <account_id> --gas=300000000000000

near call lydialabs.factory.shardnet.near unstake '{"amount": "1501013553987867946208039512"}' --accountId lydialabs.shardnet.near --gas=300000000000000

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

near view lydialabs.factory.shardnet.near get_account_unstaked_balance '{"account_id": "lydialabs.shardnet.near"}'

# Available for Withdrawal
near view <pool_id> is_account_unstaked_balance_available '{"account_id": "<account_id>"}'

near view lydialabs.factory.shardnet.near is_account_unstaked_balance_available '{"account_id": "lydialabs.shardnet.near"}'

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
near call lydialabs.factory.shardnet.near ping '{}' --accountId lydialabs.shardnet.near --gas=300000000000000
```

Visit this link: [https://explorer.shardnet.near.org/nodes/validators](https://explorer.shardnet.near.org/nodes/validators) to see if the proposal has been submitted properly, you should see something like this

![Screen Shot 2022-07-14 at 9.29.32 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2c5986a4-51cf-4bd9-90c9-b81227c684c5/Screen_Shot_2022-07-14_at_9.29.32_PM.png)

**Ping**

```bash
near call <pool_id> ping '{}' --accountId <account_id> --gas=300000000000000
```

**Example**

```bash
near call lydialabs.factory.shardnet.near ping '{}' --accountId lydialabs.shardnet.near --gas=300000000000000
```

A ping issues a new proposal and updates the staking balances for your delegators. A ping should be issued each epoch to keep reported rewards current.

**Create a cron job to run ping every epoch**

```bash
crontab -e 

# This needs to be set again else it'll look for testnet by default
NEAR_ENV=shardnet
# Current epoch is around ~2.5h, we'll invoke ping every hour for now
0 * * * * near call lydialabs.factory.shardnet.near ping '{}' --accountId lydialabs.shardnet.near --gas=300000000000000 >> $HOME/cron.log
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

![Screen Shot 2022-07-15 at 7.13.01 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/43b6c827-1284-484d-bae3-624a9b094f30/Screen_Shot_2022-07-15_at_7.13.01_PM.png)

---

# **Challenge 004**

### 004 - ****Monitor and make alerts****

In this guide we’ll be monitoring [Near Prometheus Exporter](https://github.com/masknetgoal634/near-prometheus-exporter) data scraps with Grafana dashboard. 

**Dependencies**

```bash
# Install docker
sudo apt-get update
sudo apt install docker.io
```

**Run node_exporter**

```bash
sudo docker run -dit \
    --restart always \
    --volume /proc:/host/proc:ro \
    --volume /sys:/host/sys:ro \
    --volume /:/rootfs:ro \
    --name node-exporter \
    -p 9100:9100 prom/node-exporter:latest \
    --path.procfs=/host/proc \
    --path.sysfs=/host/sys
```

**Build Near Prometheus Exporter**

```bash
git clone https://github.com/masknetgoal634/near-prometheus-exporter

cd near-prometheus-exporter

sudo docker build -t near-prometheus-exporter .
```

**Run the container**

```bash
sudo docker run -dit \
    --restart always \
    --name near-exporter \
    --network=host \
    -p 9333:9333 \
    near-prometheus-exporter:latest /dist/main -accountId lydialabs.factory.shardnet.near
```

**Configure Prometheus**

```bash
# Stay in this folder
cd near-prometheus-exporter/etc
nano prometheus/prometheus.yml

# Fill in your node IP in these targets

  - job_name: node
    scrape_interval: 5s
    static_configs:
    - targets: ['<NODE_IP_ADDRESS>:9100']

  - job_name: near-exporter
    scrape_interval: 15s
    static_configs:
    - targets: ['<NODE_IP_ADDRESS>:9333']

  - job_name: near-node
    scrape_interval: 15s
    static_configs:
    - targets: ['<NODE_IP_ADDRESS>:3030']
```

**Run Prometheus**

```bash
sudo docker run -dti \
    --restart always \
    --volume $(pwd)/prometheus:/etc/prometheus/ \
    --name prometheus \
    --network=host \
    -p 9090:9090 prom/prometheus:latest \
    --config.file=/etc/prometheus/prometheus.yml
```

**Run Grafana**

```bash
# Check user ID
id -u
1000

# Edit permission
sudo chown -R 1000:1000 grafana/*

sudo docker run -dit \
    --restart always \
    --volume $(pwd)/grafana:/var/lib/grafana \
    --volume $(pwd)/grafana/provisioning:/etc/grafana/provisioning \
    --volume $(pwd)/grafana/custom.ini:/etc/grafana/grafana.ini \
    --user 1000 \
    --network=host \
    --name grafana \
    -p 3000:3000 grafana/grafana
```

Visit <your ip>:3000 to access grafana dashboard, log in with admin/admin

![Screen Shot 2022-07-15 at 2.15.51 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7ff6152a-3e01-42b8-aa6c-02445cf76e2e/Screen_Shot_2022-07-15_at_2.15.51_PM.png)

Go to settings, data sources, Prometheus, click save and test to see if the source is properly configured.

![Screen Shot 2022-07-15 at 2.23.09 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0231a827-3483-4dfc-bede-9b095e370162/Screen_Shot_2022-07-15_at_2.23.09_PM.png)

Go to Dashboard and select Near Node Exporter Full, you should see something like this,

![Screen Shot 2022-07-15 at 2.26.33 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/23885eae-6ff2-4954-a3f0-9dd4a372f6be/Screen_Shot_2022-07-15_at_2.26.33_PM.png)

**Alerting**

You’ll need an SMTP server to send email alerts, we’ll be using [mailjet](https://www.mailjet.com/) here. Apply for an account and generate both API key and secret key.

![Screen Shot 2022-07-15 at 7.54.08 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0546166c-02ef-4d22-b513-af4fef2944e4/Screen_Shot_2022-07-15_at_7.54.08_PM.png)

Save the keys somewhere safe.

**Edit Grafana SMTP setting**

```bash
cd ~/near-prometheus-exporter/etc
nano grafana/custom.ini

[smtp]
enabled = true
host = in-v3.mailjet.com:587 
user = <your API key>
# If the password contains # or ; you have to wrap it with triple quotes. Ex """#password;"""
password = <your secret key>
;cert_file =
;key_file =
skip_verify = true
from_address = <your_email_address>
from_name = Grafana
# EHLO identity in SMTP dialog (defaults to instance_name)
;ehlo_identity = dashboard.example.com
# SMTP startTLS policy (defaults to 'OpportunisticStartTLS') 
;startTLS_policy = NoStartTLS
```

**Restart grafana container**

```bash
sudo docker ps -a

# Copy your grafana container ID
sudo docker container restart <grafana container ID>
```

Go back to your grafana dashboard, under Alerting → Contact points, enter a testing email under Addresses

![Screen Shot 2022-07-15 at 8.21.13 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4e298bb2-a042-4d3c-a615-bb280ed51194/Screen_Shot_2022-07-15_at_8.21.13_PM.png)

Click Test to see if the email sends through, successful signal looks like this.

![Screen Shot 2022-07-15 at 7.59.23 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/47dd89f5-5a5d-4d89-99ac-6fad7db51cc6/Screen_Shot_2022-07-15_at_7.59.23_PM.png)

An alert to your email will look like this

![IMG_3831.PNG](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ab790304-417c-4b92-a0da-cfedfd36df69/IMG_3831.png)

**A little more real time**

While email notifications can serve the purpose, we recommend alerting to a more real time and “louder” app, such instant messengers. Grafana has a long list of integrations,

![Screen Shot 2022-07-16 at 10.01.53 AM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5b8c9af5-3eca-4dcb-b109-0b32e6b97c94/Screen_Shot_2022-07-16_at_10.01.53_AM.png)

We’ll connect to LINE instant messenger here, first arpply for a LINE notify token [https://notify-bot.line.me/](https://notify-bot.line.me/)

![Screen Shot 2022-07-16 at 9.58.56 AM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3a1f80e4-a3b3-4f43-bd0f-cd443126c6c7/Screen_Shot_2022-07-16_at_9.58.56_AM.png)

Hook up the token to the people that need to be notified, and fill the token back in grafana. Upon an alert, you’ll get an instant message like this

![IMG_3828.PNG](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b45451fd-c124-41ca-ab51-278090b492aa/IMG_3828.png)

**Adding an alert rule**

Go to the dashboard and select a panel with a graph, you can edit a custom rule based on your preferred sensitivity settings. We’ll demonstrate adding an alert rule when the stake falls below a threshold.

![Screen Shot 2022-07-16 at 10.11.57 AM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ca677c36-e835-40e1-92cd-b7e36786f5ce/Screen_Shot_2022-07-16_at_10.11.57_AM.png)

When an alert is triggered, it’ll enter “Firing” state

![Screen Shot 2022-07-16 at 10.15.34 AM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dc2eee49-fcbb-482e-a729-e7c62d0af7ef/Screen_Shot_2022-07-16_at_10.15.34_AM.png)

You should immediately receive a notification similar to this,

![IMG_3829.PNG](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/33f80929-e26b-4cfa-890b-06422742230a/IMG_3829.png)

Its also a good idea to send out notifications to your members when an issue is resolved, 

![IMG_3830.PNG](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/839c0f48-0332-4e8a-9f82-f4451e1384cc/IMG_3830.png)

**Common RPC commands**

```bash
# Check your node version
curl -s http://127.0.0.1:3030/status | jq .version

# Check delegators and stake 
near view lydialabs.factory.shardnet.near get_accounts '{"from_index": 0, "limit": 10}' --accountId lydialabs.shardnet.near

# Check reason for validator kicked
curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' https://rpc.shardnet.near.org/ | jq -c '.result.prev_epoch_kickout[] | select(.account_id | contains ("lydialabs"))' | jq .reason

# Check blocks produced / expected 
curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' http://localhost:3030/ | jq -c '.result.current_validators[] | select(.account_id | contains ("lydialabs.factory.shardnet.near"))'
```

---

# **Challenge 005**

### 005 - Mount a node validator on AWS

This guide was already created on AWS, as seen in challenge 002. We’ll document final specs and pricing in this challenge.

**Operating System**

Ubuntu Server 22.04 LTS (HVM), SSD Volume Type

**Recommended (c5.xlarge)**

**Our spec (c5.2xlarge)**

We find the recommended specs to be sufficient for chunk production, but still upgraded it due to that fact that we’re running a few other services, including monitoring and alerting on the same instance. 

### 005 - Pricing

To create a pricing estimate, visit the [AWS calculator](https://calculator.aws/#/estimate)

c5.xlarge with 500gb SSD will run for ~$130 per month on a one year EC2 Instance Savings Plans

![Screen Shot 2022-07-16 at 4.40.07 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a1acef7b-220e-4870-8989-c9f9123f6dbf/Screen_Shot_2022-07-16_at_4.40.07_PM.png)

c5.2xlarge with 500gb SSD will run for ~$210 per month on a one year EC2 Instance Savings Plans

![Screen Shot 2022-07-16 at 4.41.24 PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/431b9edb-3b4a-4595-82db-ab1b9d94c046/Screen_Shot_2022-07-16_at_4.41.24_PM.png)

---

## Lydia Labs

Website: [https://hamado-ltd.com/](https://hamado-ltd.com)

Twitter: [https://twitter.com/mitsori_md](https://twitter.com/mitsori_md)

Github: https://github.com/dlpigpen](https://github.com/dlpigpen)

Chunk-Only Producer: [mitsorilab.factory.shardnet.near](https://explorer.shardnet.near.org/accounts/mitsorilab.factory.shardnet.near)

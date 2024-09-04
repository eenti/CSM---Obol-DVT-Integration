# Lido CSM - Obol DVT Integration
A walkthrough guide on how to integrate obol DVT into Lido's community staking module

Here's a version of the integration guide formatted for easy copying and pasting into a GitHub repository:

---

# Lido CSM – Obol Integration Guide

This guide will walk you through the process of integrating Obol Distributed Validator Technology (DVT) with the Lido Collaborative Staking Module (CSM). Obol DVT provides a decentralized and secure infrastructure for Ethereum staking, enhancing validator performance and security. Using this DVT can increase your security, and if you want to run it as a cluster, it will further reduce your 2 ETH bond. The major benifits of squad staking on CSM include: 

- Accessability
- Resilliency
- Rewards 

In this guide, we will run through Obol integration in a cluster of 4 operators, as we did for operator 69, the first CSM operator to use Obol DVT Technology. For the following guide you will need to gather a group you trust to operate with. It is important to note this guide does not outline the use of SAFE multisig wallet and for extra security and practise you should look to use one. 

## Running a DV as a Group on CSM

### Overview

Run a DV cluster as a group, where several operators run the nodes that make up the cluster. In this setup, the key shares are created using a distributed key generation process, avoiding the full private keys being stored in any one place. This approach can also be used by single operators looking to manage all nodes of a cluster while creating the key shares in a trust-minimized fashion.

You can read more about this approach [here](https://docs.obol.org/docs).

## Prerequisites

Before beginning the integration process, ensure you meet the following prerequisites:

- **Basic Knowledge**: A fundamental understanding of Ethereum nodes and validators is necessary to facilitate the integration.
  
- **Familiarity with Lido CSM**: A solid understanding of how Lido’s staking module operates is crucial for effective integration.

### Software and Tools

- **Git**: Ensure that you have Git installed on your system for version control and code management.

- **Docker**: Install Docker and make sure it is running before executing the commands. Docker is required for deploying Obol DVT nodes.

If you are unsure, please read the [Obol documentation](https://docs.obol.org/docs/start/quickstart_overview) comprehensively on how to run as a DV cluster. This will be a walkthrough of a CLI setup as the launchpad currently uses the retroactive fund address that is not accepted by the Lido DAO and CSM.

## Installation Steps

### 1. Install Prerequisite Software

#### Update Ubuntu

```bash
sudo apt update && sudo apt upgrade -y
```

#### Install Curl and Git

```bash
sudo apt install curl git -y
```

#### Install Docker and Docker Compose Plugin

**Install Docker**

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

**Remove the install script and add the current user to the Docker group**

This allows the use of Docker without `sudo`.

```bash
sudo rm -r get-docker.sh
sudo usermod -aG docker $USER
```

**Restart your shell**

This step is necessary to allow the addition of the user to the Docker group.

**Check the Docker installation**

If correctly installed, this command will output the Docker version.

```bash
docker --version
```

The script should also install the Docker Compose plugin, which you can check with the following command:

```bash
docker compose version
```

## Firewall Settings

To ensure your node communicates correctly with the network, you need to configure your firewall to allow the necessary ports.

### Required Ports

- **30303 udp/tcp**: Execution p2p ports
- **9000 udp/tcp**: Consensus p2p ports
- **3610**: Charon port

### Configure Firewall

Use the following commands to configure your firewall:

```shell
sudo ufw allow 30303
sudo ufw allow 9000
sudo ufw allow 3610
sudo ufw enable
```

**Note:** If you are running on a VPS, ensure to enable SSH with the following command to avoid locking yourself out:

```shell
sudo ufw allow ssh
```

Check the status of your firewall to verify the changes:

```shell
sudo ufw status
```

### Port Forwarding

For a local setup, you may need to port forward these ports for your device's local IP. The exact process depends on your internet provider and router model. Typically, this can be done by logging into your router's settings. You will need to do this for Charon port 3610.


---

# Obol - Create a DV with group
## Step 1: Get your ENR

### CLI

In order to prepare for a distributed key generation ceremony, you need to create an ENR for your Charon client. This ENR is a public/private key pair that allows the other Charon clients in the DKG to identify and connect to your node. If you are creating a cluster but not taking part as a node operator in it, you can skip this step.

##### Clone the repo
```shell
git clone https://github.com/ObolNetwork/charon-distributed-validator-node.git
```
##### Change directory
```shell
cd charon-distributed-validator-node/
```
##### Use docker to create an ENR. Backup the file `.charon/charon-enr-private-key`.
```shell
docker run --rm -v "$(pwd):/opt/charon" obolnetwork/charon:v1.0.0 create enr
```
You should expect to see a console output like this:

> >Created ENR private key: .charon/charon-enr-private-key
enr:-JG4QGQpV4qYe32QFUAbY1UyGNtNcrVMip83cvJRhw1brMslPeyELIz3q6dsZ7GblVaCjL_8FKQhF6Syg-O_kIWztimGAYHY5EvPgmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQKzMe_GFPpSqtnYl-mJr8uZAUtmkqccsAx7ojGmFy-FY4N0Y3CCDhqDdWRwgg4u


> ⚠️ **Warning:**
>Please make sure to create a backup of the private key at `.charon/charon-enr-private-key`. Be careful not to commit it to git! **If you lose this file you won't be able to take part in the DKG ceremony nor start the DV cluster successfully.**

> 💡 **Tip:**
>If instead of being shown your `enr` you see an error saying `permission denied`, then you may need to [update your docker permissions](../faq/errors.mdx#docker-permission-denied-error) to allow the command to run successfully.

## Step 2: Create a Cluster Using CLI

### The Following Should Be Performed by the LEADER

Collect Addresses, Configure the Cluster, Share the Invitation

Before starting the cluster creation process, you will need to collect an Ethereum address for each operator in the cluster. They will need to be able to sign messages through MetaMask with this address. (Broader wallet support will be added in the future.) With these addresses in hand, go through the cluster creation flow.

You will use the CLI to create the cluster definition file, which you will distribute manually to the operators.

The leader or creator of the cluster will prepare the `cluster-definition.json` file for the Distributed Key Generation (DKG) ceremony using the `charon create dkg` command.

Populate the `charon create dkg` command with the appropriate flags, including the `name`, `num-validators`, `fee-recipient-addresses`, `withdrawal-addresses`, and `operator-enrs` of all the operators participating in the cluster.

Run the `charon create dkg` command that generates the DKG `cluster-definition.json` file. It is important to note to change the name and number of validators you wish to run together, and when CSM is on mainnet, be sure to change the network flag as well. The fee recipient and withdrawal address should stay as they are. Make sure to read more about this [here](https://operatorportal.lido.fi/modules/community-staking-module).

**Docker Command to Create DKG Cluster Definition File:**

```bash
docker run --rm -v "$(pwd):/opt/charon" obolnetwork/charon:v1.0.0 create dkg \
  --name="Stakecat Obol Squad 69" \
  --network="holesky" \
  --num-validators=1 \
  --fee-recipient-addresses="0xE73a3602b99f1f913e72F8bdcBC235e206794Ac8" \
  --withdrawal-addresses="0xF0179dEC45a37423EAD4FaD5fCb136197872EAd9" \
  --operator-enrs="enr:-HW4QOtPBUWiFnLF1hwfxffqK-z3vmjqr8T0CwG5qk87nDFXSpLZhCZYW7jmubmlP2Typ7bgWOLvo_ABU6HDbMfW2bqAgmlkgnY0iXNlY3AyNTZrMaECWbZ5cZrmy-tK5h2r3C81PYCL5fIDf31tXzoarsfye_g,enr:-HW4QCBfh8UjiE81YcMk08cmlVdzcTwjKYyqnd8aEH8Bdco_JLRc9TJ0ygQBXKOXxw2ZRkT_Szt9aPz8VmPsIdmVrl2AgmlkgnY0iXNlY3AyNTZrMaED5XC-fmRe5k1HgyGttcTY4o7lJdHDrrheS3WsFwuLI-o,enr:-HW4QHyV1ce3zU6VIb7ETaPbob8hgRw8MIOnLf1Jryduz-psNUYeyYgP7PLKROIrI9CEODzSrIxB_ZzHYju0cBi-OdmAgmlkgnY0iXNlY3AyNTZrMaECKHwo9TrxI1-rSOxv7zq7eKGfg-ZSSgDOHjtvFvlf0dc,enr:-HW4QK1Xa3cfimvCftOAuhWPP8oGfj0WGDXbNG7diX-awInDPhoPN6D-nNzzaUzWlHXr1O6Net7gdEWNHBUR9lpMD0iAgmlkgnY0iXNlY3AyNTZrMaECz6weeL4PxNQimYkEyNYqu2RTbEGc8JteY3QrugGMV5A"
```

This command should output a file at `.charon/cluster-definition.json`. This file needs to be shared with the other operators in the cluster.

> **Note:** The `.charon` folder is hidden by default. To view it, run `ls -al .charon` in your terminal. If you are on macOS, press `Cmd + Shift + .` to view all hidden files in the Finder application.

Once every participating operator is ready, the next step is the distributed key generation amongst the operators.

- If you are not planning on operating a node and were only configuring the cluster for the operators, your journey ends here. Well done!

- If you are one of the cluster **OPERATORS**, continue to the next step.

You'll receive the `cluster-definition.json` file created by the leader/creator. You should save it in the `.charon/` folder that was created initially. (Alternatively, you can use the `--definition-file` flag to override the default expected location for this file.)

## Step 3: Run the Distributed Key Generation (DKG) ceremony

> 💡 **Tip:**
> For the DKG to complete, all operators need to be running the command simultaneously. It helps if operators can agree on a certain time or schedule a video call for them to all run the command together.

Once the creator gives you the cluster-definition.json file and you place it in a .charon subdirectory, run:
```bash
docker run --rm -v "$(pwd):/opt/charon" obolnetwork/charon:v1.0.0 dkg --publish
```
and the DKG process should begin.

### Backup Validator Keys

Make sure to take a backup of your validator_keys!!! Copy them on the same machine but also consider a offline backup

```bash
cd
mkdir .charon-backups
cd charon-distributed-validator-node
cp -r $HOME/charon-distributed-validator-node/.charon $HOME/charon-distributed-validator-node/.charon-backups
```

> ⚠️ **Warning:**
> Please make sure to create a backup of your .charon/ folder. If you lose your private keys you won't be able to start the DV cluster successfully and may risk your validator deposit becoming unrecoverable. Ensure every operator has their .charon folder securely and privately backed up before activating any validators.
>
> ❗info
> The cluster-lock and deposit-data files are identical for each operator, if lost, they can be copied from one operator to another.

Now that the DKG has been completed, all operators can start their nodes.

## Step 4: Start your Distributed Validator Node

> ❗info
> Currently, the CDVN repo configures a node for the Holesky testnet. It is possible to choose a different network (another testnet, or mainnet) by overriding the .env file. From within the charon-distributed-validator-node directory, this will be important when CSM goes to mainnet:
>.env.sample is a sample environment file that allows overriding default configuration defined in docker-compose.yml. Uncomment and set any variable to override its value.
>Setup the desired inputs for the DV, including the network you wish to operate on. Check the Charon CLI reference for additional optional flags to set.

For the sake and ease of this guide copy create a new .env and copy and paste the below into it. To do so: 

Make sure you are in your charon-distributed-validator-node directory

```bash
cd
cd charon-distributed-validator-node
```

```bash
nano .env
```

```bash
# This is a sample environment file that allows overriding default configuration defined
# in docker-compose.yml. Rename this file to `.env` and then uncomment and set any variable below.

# Overrides network for all the relevant services.
NETWORK=holesky

# Enables builder api for lodestar VC and charon services.
BUILDER_API_ENABLED=true

######### Nethermind Config #########

# Nethermind docker container image version, e.g. `latest` or `1.25.3`.
# See available tags https://hub.docker.com/r/nethermind/nethermind/tags
#NETHERMIND_VERSION=

# Nethermind host exposed ports
#NETHERMIND_PORT_P2P=
#NETHERMIND_PORT_HTTP=
#NETHERMIND_PORT_ENGINE=

######### Lighthouse Config #########

# Lighthouse beacon node docker container image version, e.g. `latest` or `v4.6.0`.
# See available tags https://hub.docker.com/r/sigp/lighthouse/tags.
#LIGHTHOUSE_VERSION=

# Lighthouse beacon node host exposed ports
#LIGHTHOUSE_PORT_P2P=

# Checkpoint sync url used by lighthouse to fast sync.
# See available options https://eth-clients.github.io/checkpoint-sync-endpoints/.
#LIGHTHOUSE_CHECKPOINT_SYNC_URL=

######### Lodestar Config #########

# Lodestar validator client docker container image version, e.g. `latest` or `v1.15.1`.
# See available tags https://hub.docker.com/r/chainsafe/lodestar/tags
#LODESTAR_VERSION=

# Override prometheus metrics port for lodestar validator client.
#LODESTAR_PORT_METRICS=

######### Charon Config #########

# Charon docker container image version, e.g. `latest` or `v1.0.0`.
# See available tags https://hub.docker.com/r/obolnetwork/charon/tags.
#CHARON_VERSION=

# Define custom relays. One or more ENRs or an http URL that return an ENR. Use a comma separated list excluding spaces.
#CHARON_P2P_RELAYS=

# Connect to one or more external beacon nodes. Use a comma separated list excluding spaces.
#CHARON_BEACON_NODE_ENDPOINTS=

# Override the charon logging level; debug, info, warning, error.
#CHARON_LOG_LEVEL=

# Override the charon logging format; console, logfmt, json. Grafana panels require logfmt.
#CHARON_LOG_FORMAT=

# Advertise a custom external DNS hostname or IP address for libp2p peer discovery.
#CHARON_P2P_EXTERNAL_HOSTNAME=

# Loki log aggregation server addresses. Disable loki log aggregation by setting an empty address.
#CHARON_LOKI_ADDRESSES=

# Docker network of running charon node. See `docker network ls`.
#CHARON_DOCKER_NETWORK=

# Charon host exposed ports
#CHARON_PORT_P2P_TCP=

######### MEV-Boost Config #########

# MEV-Boost docker container image version, e.g. `latest` or `1.6`.
# Note that mev-boost tag 1.6.1a3 supports the holesky network.
#MEVBOOST_VERSION=

# Comma separated list of MEV-Boost relays.
# You can choose public relays from https://enchanted-direction-844.notion.site/6d369eb33f664487800b0dedfe32171e?v=d255247c822c409f99c498aeb6a4e51d.
MEVBOOST_RELAYS=https://0xafa4c6985aa049fb79dd37010438cfebeb0f2bd42b115b89dd678dab0670c1de38da0c4e9138c9290a398ecd9a0b3110@boost-relay-holesky.flashbots.net,https://0xaa58208899c6105603b74396734a6263cc7d947f444f396a90f7b7d3e65d102aec7e5e5291b27e08d02c50a050825c2f@holesky.titanrelay.xyz,https://0xb1559beef7b5ba3127485bbbb090362d9f497ba64e177ee2c8e7db74746306efad687f2cf8574e38d70067d40ef136dc@relay-stag.ultrasound.money,https://0xab78bf8c781c58078c3beb5710c57940874dd96aef2835e7742c866b4c7c0406754376c2c8285a36c630346aa5c5f833@holesky.aestus.live

######### Monitoring Config #########

# Grafana docker container image version, e.g. `latest` or `9.4.3`.
# See available tags https://github.com/grafana/grafana/releases.
#GRAFANA_VERSION=

# Grafana host exposed port
#MONITORING_PORT_GRAFANA=

# Prometheus docker container image version, e.g. `latest` or `v2.42.0`.
# See available tags https://github.com/prometheus/prometheus/releases.
#PROMETHEUS_VERSION=

######### Voluntary Exit Config #########

# This applies to compose-voluntary-exit.yml only

# Cluster wide consistent exit epoch. Set to latest for fork version, see `curl $BEACON_NODE/eth/v1/config/fork_schedule`
#EXIT_EPOCH=

######### Debug Config #########

# This applies to compose-debug.yml only

# Prometheus Node exporter docker container image version, e.g. `latest` or `1.5.0`.
# See available tags https://hub.docker.com/r/bitnami/node-exporter/tags.
#NODE_EXPORTER_VERSION=

# Jaeger docker container image version, e.g. `latest` or `1.42.0`.
# See available tags https://hub.docker.com/r/jaegertracing/all-in-one/tags.
#JAEGER_VERSION=

# Jaeger host exposed port for HTTP query.
#MONITORING_PORT_JAEGER=

# Grafana Loki docker container image version, e.g. `latest` or `2.8.2`.
# See available tags https://hub.docker.com/r/grafana/loki/tags.
#LOKI_VERSION=

# Loki host exposed port
#MONITORING_PORT_LOKI=
```
> Use `crtl+o` to save the .env then `ctrl+x` to exit the .env file.

To run our validator node, we must first run and fully sync and Execution layer client and consensus layer client. By default Charon is set to sync execution layer client (geth) and a consensus layer client (lighthouse).

While still in the the working folder charon-distributed-validator-cluster cp command:
```bash
docker compose up -d
```
## Step 5: Upload Deposit Data to CSM

The next step should be taken by the cluster leader.

1. Go to [https://csm.testnet.fi/](https://csm.testnet.fi/)
2. Select **Become a Node Operator** and then **Create a Node Operator**.
3. On the CSM Widget, upload your deposit data file and select the corresponding bond type (ETH, stETH, wstETH), and provide the desired bond amount.
4. You will then need to save your `deposit.json` to your local machine to drag and drop, or alternatively, use the following command and copy and paste the output into the ‘Upload Deposit Data’ box on the CSM front end:

    ```bash
    cat ~/charon-distributed-validator-node/.charon/deposit_data.json
    ```

5. Finally, select **Create Node Operator**, sign the transaction with your connected wallet, and you are all set.

Now you just need to wait for the Lido CSM to deposit your validator keys (using your deposit data file). Give some time for the keys to be deposited, and then you will be able to click the Beaconchain link to view your validator.

🎉Congratulations, you are now running a CSM Obol DVT Validator.🎉



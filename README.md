# CSM---Obol-DVT-Integration
A walkthrough guide on how to integrate obol DVT into Lido's community staking module

Here's a version of the integration guide formatted for easy copying and pasting into a GitHub repository:

---

# Lido CSM – Obol Integration Guide

This guide will walk you through the process of integrating Obol Distributed Validator Technology (DVT) with the Lido Collaborative Staking Module (CSM). Obol DVT provides a decentralized and secure infrastructure for Ethereum staking, enhancing validator performance and security. Using this DVT can increase your security, and if you want to run it as a cluster, it will further reduce your 2 ETH bond.

In this guide, we will run through Obol integration in a cluster of 4 nodes, as we did for operator 69, the first CSM operator to use Obol DVT Technology. Obol is also releasing a 1% retroactive consensus fund, which can also be contributed to through CSM using a custom rewards address with a splitter contract. Instructions on how to contribute to this will follow.

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

For a local setup, you may need to port forward these ports for your device's local IP. The exact process depends on your internet provider and router model. Typically, this can be done by logging into your router's settings.


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
>
>Please make sure to create a backup of the private key at `.charon/charon-enr-private-key`. Be careful not to commit it to git! **If you lose this file you won't be able to take part in the DKG ceremony nor start the DV cluster successfully.**
>
> 💡 **Tip:**
>
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

> ⚠️ **Warning:**
> Please make sure to create a backup of your .charon/ folder. If you lose your private keys you won't be able to start the DV cluster successfully and may risk your validator deposit becoming unrecoverable. Ensure every operator has their .charon folder securely and privately backed up before activating any validators.


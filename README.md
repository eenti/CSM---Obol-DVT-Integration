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

This guide now includes instructions for setting up firewall rules and port forwarding, ensuring your node is properly configured for network operations. You can copy and paste this directly into your GitHub repository for clear and comprehensive documentation. Let me know if there's anything else you'd like to add or modify!

```markdown
## Step 1: Get your ENR

In order to prepare for a distributed key generation ceremony, you need to create an ENR for your Charon client. This ENR is a public/private key pair that allows the other Charon clients in the DKG to identify and connect to your node. If you are creating a cluster but not taking part as a node operator in it, you can skip this step.

```shell
# Clone the repo
git clone https://github.com/ObolNetwork/charon-distributed-validator-node.git

# Change directory
cd charon-distributed-validator-node/

# Use docker to create an ENR. Backup the file `.charon/charon-enr-private-key`.
docker run --rm -v "$(pwd):/opt/charon" obolnetwork/charon:v1.0.0 create enr
```

You should expect to see a console output like this:

```
Created ENR private key: .charon/charon-enr-private-key
enr:-JG4QGQpV4qYe32QFUAbY1UyGNtNcrVMip83cvJRhw1brMslPeyELIz3q6dsZ7GblVaCjL_8FKQhF6Syg-O_kIWztimGAYHY5EvPgmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQKzMe_GFPpSqtnYl-mJr8uZAUtmkqccsAx7ojGmFy-FY4N0Y3CCDhqDdWRwgg4u
```

**Warning:**

Please make sure to create a backup of the private key at `.charon/charon-enr-private-key`. Be careful not to commit it to git! **If you lose this file you won't be able to take part in the DKG ceremony nor start the DV cluster successfully.**

**Tip:**

If instead of being shown your `enr` you see an error saying `permission denied`, then you may need to [update your docker permissions](../faq/errors.mdx#docker-permission-denied-error) to allow the command to run successfully.
```


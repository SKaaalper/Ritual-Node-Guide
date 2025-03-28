# Ritual Node Setup Guide

![image](https://github.com/user-attachments/assets/e35e183e-256a-49d7-8279-5c585e9ca322)

Infernet is a lightweight library for bridging off-chain compute on-chain. With the help of Infernet, smart contract developers can request computation to be executed off-chain by Infernet Nodes and delivered to their on-chain smart contracts via the Infernet SDK.

## Recommended Hardware Requirements (using Contabo VPS 2)
- **CPU**: 4 modern vCPU cores
- **Memory (RAM)**: 16GB
- **Storage**: 500GB IOPS-optimized SSD

## Prerequisites
- EVM Wallet: Must hold ETH tokens on the Base Mainnet
- Ensure the wallet has at least **$15–25** worth of ETH to cover gas fees
- Use Burner Wallet

## Node Installation:

### 1. Install Dependencies:
1. Update the system and install necessary packages:
```
sudo apt update && sudo apt upgrade -y
sudo apt -qy install curl git jq lz4 build-essential screen
```

### 2. Install docker & Docker Compose (If you already have it, just skip):
```
# Add Docker’s GPG key and repo
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker & Compose Plugin
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
- Check Version:
```
docker --version
docker compose version
```
- ✅ If the version is `v2.20.2` or above, you're good to go.

### 3. Add User to Docker Group:
```
sudo usermod -aG docker $USER
```

### 4. Verify Docker Installation:
```
docker run hello-world
```

### 5. Clone `Ritual Repo` & `Deploy Initial Container`:

1. Clone the Repo:
```
git clone https://github.com/ritual-net/infernet-container-starter
cd infernet-container-starter
```
2. Start a screen session (for background process):
```
screen -S ritual
```
3. Deploy the container:
```
project=hello-world make deploy-container
```
4. Detach from the screen: **Press Ctrl + A, Then Press D**

### 6. Update Configuration Files:

1. Edit the main config file:
```
nano ~/infernet-container-starter/deploy/config.json
```
2. Update the following variables:
  - **"rpc_url"**: `https://mainnet.base.org/`
  - **"private_key"**: `your new empty EVM wallet's private key (with $10–15 ETH on Base)`
  - **"registry"**: `0x3B1554f346DFe5c482Bb4BA31b880c1C18412170`
  - **"sleep"**: `3`
  - **"starting_sub_id"**: `160000`
  - **"batch_size"**: `800`
  - **"sync_period"**: `30`
  - **"trail_head_blocks"**: `3`
 
  ![image](https://github.com/user-attachments/assets/55f37cd4-1f09-427f-b8ad-4253010e093b)

- **Ensure proper JSON formatting (quotes, commas, etc.)**

Save and exit: `Ctrl + X`, then `press Y`, then `Enter`.

3. Repeat the same changes in:
```
nano ~/infernet-container-starter/projects/hello-world/container/config.json
```

![image](https://github.com/user-attachments/assets/c4e62833-798c-4965-9c51-197d275e7deb)


4. Update registry address in this file:
```
nano ~/infernet-container-starter/projects/hello-world/contracts/script/Deploy.s.sol
```
- Replace `address registry = ...` with: `0x3B1554f346DFe5c482Bb4BA31b880c1C18412170`

![image](https://github.com/user-attachments/assets/741d6268-2fb7-4868-a430-51ff13ef444a)

5. Update Makefile:
```
nano ~/infernet-container-starter/projects/hello-world/contracts/Makefile
```
- **sender**: `your private key`
- **RPC_URL**: `https://mainnet.base.org/`

![image](https://github.com/user-attachments/assets/e8c95edd-d216-4f99-9ae8-81bd00039f08)

6. Update Docker Compose version:
```
nano ~/infernet-container-starter/deploy/docker-compose.yaml
```
- image: ritualnetwork/infernet-node: `1.4.0`

![image](https://github.com/user-attachments/assets/983b25e6-c3dd-4925-91ac-e42b75cf5653)

### 7. Restart the Container:

1. Re-enter screen session:
```
screen -r ritual
```

2. Stop existing containers:
```
docker compose -f ~/infernet-container-starter/deploy/docker-compose.yaml down
```

3. Start containers again:
```
docker compose -f ~/infernet-container-starter/deploy/docker-compose.yaml up
```
- Detach from the screen: **Press Ctrl + A, Then Press D**

### 8. Install Foundry:
```
mkdir foundry
cd foundry
curl -L https://foundry.paradigm.xyz | bash
source ~/.bashrc
```
```
foundryup
```

![image](https://github.com/user-attachments/assets/f358d7a0-0af9-42a0-a3a2-2cb3cd49e22f)


### 9. Install Dependencies:

1. Go to contracts folder:
```
cd ~/infernet-container-starter/projects/hello-world/contracts
```

2. Install `forge-std`:
```
forge install --no-commit foundry-rs/forge-std
```
- If you get git `submodule exited with code 128 error`, run:
```
rm -rf lib/forge-std
forge install --no-commit foundry-rs/forge-std
```

3. Install `infernet-sdk`:
```
forge install --no-commit ritual-net/infernet-sdk
```
- If error persists, run:
```
rm -rf lib/infernet-sdk
forge install --no-commit ritual-net/infernet-sdk
```

### 10. Deploy and Interact with the Contract:
    
1. Re-enter the screen session:
```
screen -r ritual
```

2. Stop container:
```
docker compose -f ~/infernet-container-starter/deploy/docker-compose.yaml down
```

3. Restart container:
```
docker compose -f ~/infernet-container-starter/deploy/docker-compose.yaml up
```
- Detach from the screen: **Press Ctrl + A, Then Press D**

4. Navigate Back to:
```
cd ~/infernet-container-starter
```

5 Deploy SaysGM contract:
```
project=hello-world make deploy-contracts
```

![image](https://github.com/user-attachments/assets/0db3a735-3bb5-4e04-82ad-76f84caedb78)

- Save the returned `contract address`.

6. Update `CallContract.s.sol`:
```
nano ~/infernet-container-starter/projects/hello-world/contracts/script/CallContract.s.sol
```

![image](https://github.com/user-attachments/assets/dc04bd87-5ea4-46f3-840b-14cd8e8b6426)

- Replace the contract address in `SaysGM(<contract_address>)` with the one from the previous step.

7. Call the contract:
```
project=hello-world make call-contract
```
![image](https://github.com/user-attachments/assets/ed789a30-f897-4fa2-acf0-9ec4a1fa59e3)

### 11. Final Checks & Manual Transaction:

- Re-enter screen again:
```
screen -r ritual
```

![image](https://github.com/user-attachments/assets/c97e036a-c785-40e9-96b0-3507559ae3d8)

- If logs show block data streaming in, the node is working. Detach with **Press Ctrl + A, Then Press D**

- To finish setup manually:
- Go to: https://basescan.org/address/0x8d871ef2826ac9001fb2e33fdd6379b6aabf449c
   - Connect your wallet (used for the node setup)
   - Go to function #8 registerNode → enter your wallet address and click Write.
   - Wait 24+ hours, then go to #1 activateNode → and sign that transaction too.
 
### Now, Go to Guild:
1. Join the Guild and Get Your Role, Use your funding wallet to join the Ritual Guild: https://guild.xyz/ritual
2. Link your Node Wallet address In guild
3. Once you've joined, you’ll receive the Node Runner Role.

![image](https://github.com/user-attachments/assets/428541ba-0470-4b17-a5fd-b353396b3770)

Alternatively, you can follow the official guide to set up and run the node manually: [Official Node Guide](https://ritual.academy/nodes/setup/)

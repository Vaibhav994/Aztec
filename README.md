# Aztec
A step by step guide on How to Run `Sequencer Node` on Aztec Network Testnet & Earn `Apprentice` & `Guardian`Role.

---

## 1️⃣ Install Dependecies
* Update packages:
```bash
sudo apt-get update && sudo apt-get upgrade -y
```

* Install Packages:
```bash
sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev  -y
```

* Install Docker:
```bash
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


sudo apt update
apt install docker.io
sudo apt install docker-compose-plugin

# Test Docker
sudo docker run hello-world
sudo systemctl enable docker
sudo systemctl restart docker
```

## 2️⃣ Install Aztec Tools
```bash
bash -i <(curl -s https://install.aztec.network)
```
```bash
echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc

source ~/.bashrc
```

* Check if you installed successfully:
```bash
aztec
```

* Update Aztec
```bash
aztec-up latest
```

## 3️⃣ Obtain RPC URLs and Generate Ethereum Keys
* You can get Sepolia `Geth` and `Beacon` RCP from TF Team at discounted price. (Limited Slots only!)
* Get an EVM Wallet with `Private Key` and `Public Address` saved.

* Fund your Ethereum Wallet with [`ETH Sepolia`](https://cloud.google.com/application/web3/faucet/ethereum/sepolia).

## 4️⃣ Find IP and Enable Firewall & Open Ports
```bash
curl ifconfig.me
```
* Save it

* Enable Firewall & Open Ports
```console
# Firewall
ufw allow 22
ufw allow ssh
ufw enable

# Sequencer
ufw allow 40400
ufw allow 8080
```

## 5️⃣ Run Sequencer Node
* Open TMUX
```bash
tmux new -s aztec
```

* Run Node
```
aztec start --node --archiver --sequencer \
  --network alpha-testnet \
  --l1-rpc-urls RPC_URL  \
  --l1-consensus-host-urls BEACON_URL \
  --sequencer.validatorPrivateKey 0xYourPrivateKey \
  --sequencer.coinbase 0xYourAddress \
  --p2p.p2pIp IP
  --p2p.maxTxPoolSize 1000000000
```

Replace the following variables before you Run Node:
* `RPC_URL` & `BEACON_URL`: Step 4
* `0xYourPrivateKey`: Your EVM wallet private key starting with `0x...`
* `0xYourAddress`: Your EVM wallet public address starting with `0x...`
* `IP`: Your server IP (Step 7)

### Optional Commands:
**Tmux Commands:**
* Minimze Tmux: `Ctrl` + `B` + `D`
* Return to Tmux: `tmux attach -t aztec`
* Kill Tmux (when inside): `Ctrl`+`B` + `X`
* Kill Tmux (when outside): `tmux kill-pane -t aztec`

## 6️⃣ Check Sync Status
* After entering the command, your node starts running, It takes a few minutes for your node to get synced.

* Check the latest synced block number of your sequencer:
```
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' \
http://localhost:8080 | jq -r ".result.proven.number"
```
* Check the latest block number of Aztec network: https://aztecscan.xyz/

## 7️⃣ Verify Node's Peer ID:
**Find your Node's Peer ID:**
```bash
sudo docker logs $(docker ps -q --filter ancestor=aztecprotocol/aztec:latest | head -n 1) 2>&1 | grep -i "peerId" | grep -o '"peerId":"[^"]*"' | cut -d'"' -f4 | head -n 1

```
* This reveals your Node's Peer ID, Now search it on [Nethermind Explorer](https://aztec.nethermind.io/)
* Note: It might take some hours for your node to show up in Nethermind Explorer after it fully synced.

## 8️⃣ Save P2P Private Key :
**Find your Node's P2P private key:**
```bash
nano .aztec/alpha-testnet/data/p2p-private-key
```
* This reveals your Node's P2P Private key, Save it for future use.
* You can use it to re-run your node on another vps with same Peer ID.

---



## 🔃 Update Sequencer Node
1️⃣ Stop Node:
```console
Ctrl + c
```

2️⃣ Update Node:
```bash
aztec-up latest
```


3️⃣ Re-run Node

Return to [Step 5: Run Sequencer Node](https://github.com/Vaibhav994/Aztec/blob/main/README.md#5%EF%B8%8F%E2%83%A3-run-sequencer-node) to re-run your node

---

## Upgrade Aztec Node 
 * Stop Aztec Node with `CTRL + C`

1️⃣ Upgrade to v1.1.0
```
aztec-up 1.1.0
```

* Clear Data and Worldstate
```
rm -rf /tmp/aztec-world-state-*
rm -rf ~/.aztec/alpha-testnet/data
```

 * Update your startup command:
   + Replace --sequencer.validatorPrivateKey ➡️=>>> with `--sequencer.validatorPrivateKeys` (note the plural)
   + Provide a comma-separated list of private keys to run multiple validators (eg: upto 10).
     
 * (Optional) Set a --sequencer.publisherPrivateKey
   + This address will post transactions.
   + Only this address needs to be funded with Sepolia ETH if you’re running multiple validators.

2️⃣ In your current shell use; 
```
export COINBASE=0xaddress
export LOG_LEVEL=debug
export P2P_MAX_TX_POOL_SIZE=1000000000
```

3️⃣ Start Aztec by updating [Step 5: Run Sequencer Node](https://github.com/Vaibhav994/Aztec/blob/main/README.md#5%EF%B8%8F%E2%83%A3-run-sequencer-node) command or use this command (e.g) 
```
aztec start \
  --network alpha-testnet \
  --l1-rpc-urls " " \
  --l1-consensus-host-urls " " \
  --sequencer.validatorPrivateKeys "      " \
  --p2p.p2pIp "      " \
  --archiver \
  --node \
  --sequencer
```


---
## Get Apprentice Discord Role:
Go to the discord channel :[operators| start-here](https://discord.com/channels/1144692727120937080/1367196595866828982/1367323893324582954) and type `/operator start` then follow the prompts.

**1️⃣ Get the latest proven block number:**
```bash
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' \
http://localhost:8080 | jq -r ".result.proven.number"
```
* Save this block number for the next steps
* Example output: 20905

**2️⃣ Generate your sync proof**
```bash
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getArchiveSiblingPath","params":["BLOCK_NUMBER","BLOCK_NUMBER"],"id":67}' \
http://localhost:8080 | jq -r ".result"
```
* Replace 2x `BLOCK_NUMBER` with your number

**3️⃣ Register with Discord**
* Type the following command in this Discord server: `/operator start`
* After typing the command, Discord will display option fields that look like this:
* `address`:            Your validator address (Ethereum Address)
* `block-number`:      Block number for verification (Block number from [Step 1](https://github.com/OneEyeKing001/aztec?tab=readme-ov-file#1-install-dependecies))
* `proof`:             Your sync proof (base64 string from [Step 2](https://github.com/OneEyeKing001/aztec?tab=readme-ov-file#2-install-aztec-tools))

Then you'll get your `Apprentice` Role.


## Get Guardian Discord Role:
Guardian Role snapshot takes place every 5-7 days, start you node today and wait for next snapshot.

**After Snapshot**

Go to the discord channel :[operators| start-here](https://discord.com/channels/1144692727120937080/1367196595866828982/1367323893324582954) and type `/check ip` then Enter your `IP` and `node_eth_address` 

Then you'll get your `Guardian` Role

* `IP`:                      Vps Ip you get in [Step 4](https://github.com/Vaibhav994/Aztec/blob/main/README.md#4%EF%B8%8F%E2%83%A3-find-ip-and-enable-firewall--open-ports)

* `node_eth_address `:       ETH Public address you generated in [Step 3](https://github.com/Vaibhav994/Aztec/blob/main/README.md#3%EF%B8%8F%E2%83%A3-obtain-rpc-urls-and-generate-ethereum-keys)

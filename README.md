# Aligned-Layer
### Prerequisites
- **Operating System:** Linux (Ubuntu 22.04 or higher is recommended)
- **Hardware Requirements:** At least 4 CPU cores, 8 GB RAM, and 500 GB SSD storage
- **Dependencies:** `git`, `curl`, `wget`, `jq`, `build-essential`, `go`

### 1. Update System and Install Dependencies
Update your system packages and install necessary dependencies:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install make curl git wget jq build-essential -y
```

### 2. Install Go
Download and install Go, which is required to build the node:
```bash
ver="1.21.5"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

### 3. Clone and Build Aligned Layer Node
Clone the Aligned Layer repository and build the node:
```bash
cd $HOME
git clone --depth 1 --branch v0.0.2 https://github.com/yetanotherco/aligned_layer_tendermint
cd aligned_layer_tendermint/cmd/alignedlayerd
go build
chmod +x alignedlayerd
sudo mv alignedlayerd /usr/local/bin/
```

### 4. Initialize the Node
Initialize your node with a custom moniker (replace `NodeName`):
```bash
alignedlayerd init NodeName --chain-id alignedlayer
```

### 5. Download Genesis and Addrbook Files
Download the genesis and addrbook files needed to sync with the network:
```bash
wget https://raw.githubusercontent.com/yetanotherco/aligned_layer/main/genesis.json -O $HOME/.alignedlayer/config/genesis.json
wget https://raw.githubusercontent.com/yetanotherco/aligned_layer/main/addrbook.json -O $HOME/.alignedlayer/config/addrbook.json
```

### 6. Configure Persistent Peers
Add the necessary peer information to the configuration file:
```bash
PEERS="peer_address_1,peer_address_2"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/.alignedlayer/config/config.toml
```

### 7. Start the Node
Start your node using `systemd` to ensure it runs in the background and restarts on failure:
```bash
sudo tee /etc/systemd/system/alignedlayerd.service > /dev/null <<EOF
[Unit]
Description=Aligned Layer Node
After=network-online.target

[Service]
User=$USER
ExecStart=/usr/local/bin/alignedlayerd start
Restart=on-failure
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable alignedlayerd
sudo systemctl start alignedlayerd
```

### 8. Monitor Node Logs
Check if the node is running properly by monitoring the logs:
```bash
journalctl -u alignedlayerd -f
```

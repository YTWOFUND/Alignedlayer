# Alignedlayer

### Alignedlayer node Installation Instructions.

[Official documentation](https://docs.alignedlayer.com)

System requirements:</br>
CPU: Quad Core or larger AMD or Intel (amd64) CPU
RAM:32GB RAM
SSD:1TB NVMe Storage
100MBps bidirectional internet connection
OS: Ubuntu 20.04 or 22.04</br>

You can take a weaker server

### Network configurations: </br>
Outbound - allow all traffic </br>
Inbound - open the following ports :</br>
1317 - REST </br>
26657 - TENDERMINT_RP </br>
26656 - Cosmos </br>
Running as a Provider? Add these specific ports: </br>
22221 - provider port </br>
22231 - provider port </br>
22241 - provider port </br>

### Installing the Babylon Node

1. Preparing the server/Required packages installation</br>
```
sudo apt update
sudo apt upgrade -y
sudo apt-get install libclang-dev
sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source $HOME/.cargo/env
curl https://get.ignite.com/cli | bash
sudo mv ignite /usr/local/bin/
```
### Go installation.
```
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```

### Download and build binaries
```
rm -rf $HOME/aligned_layer_tendermint
git clone https://github.com/yetanotherco/aligned_layer_tendermint.git
cd $HOME/aligned_layer_tendermint
git checkout 98643167990f8a597b331ddd879e079bafb25b08
make build-linux
```

# Config and init app
```
alignedlayerd init "$MONIKER" --chain-id alignedlayer
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${ALIGNEDLAYER_PORT}657\"|" $HOME/.alignedlayer/config/client.toml
```

# Download genesis and addrbook
```
wget -O $HOME/.alignedlayer/config/genesis.json https://testnet-files.itrocket.net/alignedlayer/genesis.json
wget -O $HOME/.alignedlayer/config/addrbook.json https://testnet-files.itrocket.net/alignedlayer/addrbook.json
```

# Set seeds and peers
```
SEEDS="d1a8816c1c5800b352c2a1eb0e7a156bce34ae9f@alignedlayer-testnet-seed.itrocket.net:50656"
PEERS="144c2d4fbbaf54dda837bfbc88b688fb2f02c92f@alignedlayer-testnet-peer.itrocket.net:50656,2567ea5aed4bba4e3062a1072a8f1e7fb4e4497c@65.109.85.36:26656,51ca4087558ebe93a16e3f1e84a969d30e7a91f1@95.216.245.35:26656,4093bf12076818a82f9fc1c75dc974e1d93daf44@195.201.30.159:26656,df898a791ae0aa21c1e2029c90ff8275104860d8@37.60.248.171:26656,18e1adeadb8cc596375e4212288fcd00690df067@213.199.48.195:26656,7292de855372480ae23dbcaf94d36ead187cf6a8@194.163.143.206:50656,a1d6d9569789a7a8765f0a4899439819f07755d4@213.133.103.213:26656,afeea4cd47aa80504adbdaa8aa019864e291de55@[2a03:cfc0:8000:13::b910:277f]:13356,5be66a14f474ea7c8abe6d576758fa14d1289793@154.12.228.190:26656,6b89aeeb3d1a4352f684f38545368e78889249e5@213.199.44.1:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.alignedlayer/config/config.toml
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.alignedlayer/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.alignedlayer/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.alignedlayer/config/app.toml
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.0001stake"|g' $HOME/.alignedlayer/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.alignedlayer/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.alignedlayer/config/config.toml
```

# Create service file
```
sudo tee /etc/systemd/system/alignedlayerd.service > /dev/null <<EOF
[Unit]
Description=Alignedlayer node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.alignedlayer
ExecStart=$(which alignedlayerd) start --home $HOME/.alignedlayer
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable alignedlayerd
```

# Reset and download snapshot
```
alignedlayerd tendermint unsafe-reset-all --home $HOME/.alignedlayer
if curl -s --head curl https://testnet-files.itrocket.net/alignedlayer/snap_alignedlayer.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://testnet-files.itrocket.net/alignedlayer/snap_alignedlayer.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.alignedlayer
    else
  echo no have snap
fi
```

# enable and start service
```
sudo systemctl start alignedlayerd
sudo systemctl start alignedlayerd
```

### Becoming a Validator

# Create wallet key new
```
alignedlayerd keys add $WALLET
```

(OPTIONAL) RECOVER EXISTING KEY
```
alignedlayerd keys add $WALLET --recover
```

# check sync status, once your node is fully synced, the output from above will print "false"
```
alignedlayerd status 2>&1 | jq
```

### We receive tokens from the tap in the [discord](https://discord.gg/alignedlayer)
```
faucet not work
```

# before creating a validator, you need to fund your wallet and check balance
```
alignedlayerd q bank balances $WALLET_ADDRESS 
```
# Create validator
```
alignedlayerd tx staking create-validator \
--amount 1000000stake \
--from $WALLET \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(alignedlayerd tendermint show-validator) \
--moniker "$MONIKER" \
--identity "FFB0AA51A2DF5955 " \
--details "I love YTWO❤️" \
--chain-id alignedlayer \
--fees 50stake \
-y 
```

### Update
```
No update

Current network:alignedlayer
Current version:v1.0.0-testnet
```

### Useful commands

Check balance
```
alignedlayerd q bank balances $WALLET_ADDRESS 
```

CHECK SERVICE LOGS
```
sudo journalctl -u alignedlayerd -f
```

RESTART SERVICE
```
sudo systemctl restart alignedlayerd
```

GET VALIDATOR INFO
```
alignedlayerd status 2>&1 | jq
```

DELEGATE TOKENS TO YOURSELF
```
alignedlayerd tx staking delegate $(alignedlayerd keys show $WALLET --bech val -a) 1000000stake --from $WALLET --chain-id alignedlayer --fees 50stake -y 
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop alignedlayerd && sudo systemctl disable alignedlayerd && sudo rm /etc/systemd/system/alignedlayerd.service && sudo systemctl daemon-reload && rm -rf $HOME/.alignedlayerd && rm -rf alignedlayerd && sudo rm -rf $(which alignedlayerd) 
```

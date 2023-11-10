# Install Guide Realio

## [Website](https://www.realio.fund/) | [Discord](https://t.co/XfBw6uOFSq) | [Twitter](https://twitter.com/realio_network) | :satellite:[Explorer](https://explorer.moonbridge.team/realio)

**Chain ID:** realionetwork_3301-1 | **Latest Version:** v0.8.3 | **Custom Port:** 152

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
REALIO_PORT=152
```

## Update system and install tools

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget build-essential git jq tar pkg-config libssl-dev liblz4-tool ncdu bashtop -y
```

## Install GO

```bash
cd $HOME
version="1.21.4"
wget "https://golang.org/dl/go$version.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$version.linux-amd64.tar.gz"
rm "go$version.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Download and install

```bash
cd $HOME
git clone https://github.com/realiotech/realio-network.git
cd realio-network
git checkout v0.8.3
make install
realio-networkd version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
realio-networkd config node tcp://localhost:${REALIO_PORT}57
realio-networkd config chain-id realionetwork_3301-1
realio-networkd config keyring-backend file
realio-networkd init $MONIKER --chain-id realionetwork_3301-1

# Download genesis and addrbook
curl -Ls https://snapshots.moonbridge.team/mainnet/realio/genesis.json > $HOME/.realio-network/config/genesis.json
curl -Ls https://snapshots.moonbridge.team/mainnet/realio/addrbook.json > $HOME/.realio-network/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.realio-network/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0ario\"|" $HOME/.realio-network/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.realio-network/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.realio-network/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.realio-network/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.realio-network/config/app.toml

# Disable indexer
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.realio-network/config/config.toml

# Enable Prometheus
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.realio-network/config/config.toml

# Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${REALIO_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${REALIO_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${REALIO_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${REALIO_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${REALIO_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${REALIO_PORT}66\"%" $HOME/.realio-network/config/config.toml
sed -i -e "s%:1317%:${REALIO_PORT}17%g; s%:8080%:${REALIO_PORT}80%g; s%:9090%:${REALIO_PORT}90%g; s%:9091%:${REALIO_PORT}91%g; s%:8545%:${REALIO_PORT}45%g; s%:8546%:${REALIO_PORT}46%g; s%:6065%:${REALIO_PORT}65%g" $HOME/.realio-network/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/realio-networkd.service > /dev/null <<EOF
[Unit]
Description=Realio Node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.realio-network
ExecStart=$(which realio-networkd) start --home $HOME/.realio-network
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable realio-networkd
```

## Start service and check the logs

```bash
sudo systemctl start realio-networkd && sudo journalctl -u realio-networkd -f --no-hostname -o cat
```

## Create wallet

```bash
realio-networkd keys add wallet
```

## Check wallet balance

```bash
realio-networkd q bank balances $(realio-networkd keys show wallet -a)
```

## Create validator

```bash
realio-networkd tx staking create-validator \
  --amount 1000000ario \
  --pubkey $(realio-networkd tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id realionetwork_3301-1 \
  --commission-rate 0.05 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0ario \
  -y
```

## Delete node

```bash
sudo systemctl stop realio-networkd
sudo systemctl disable realio-networkd
sudo rm -rf /etc/systemd/system/realio-networkd.service
sudo systemctl daemon-reload
sudo rm -f $(which realio-networkd) 
sudo rm -rf $HOME/.realio-network 
sudo rm -rf $HOME/realio-network
```

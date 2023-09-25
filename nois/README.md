# Install Guide Nois

## [Website](https://nois.network/) | [Discord](https://discord.gg/dHdpwtEb6F) | [Twitter](https://twitter.com/NoisRNG) | :satellite:[Explorer](https://explorer.moonbridge.team/nois-mainnet)

**Chain ID:** nois-1 | **Latest Version:** v1.0.3 | **Custom Port:** 153

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
NOIS_PORT=153
```

## Update system and install tools

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget build-essential git jq tar pkg-config libssl-dev liblz4-tool ncdu bashtop -y
```

## Install GO

```bash
cd $HOME
version="1.20.5"
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
git clone https://github.com/noislabs/noisd.git
cd noisd
git checkout v1.0.3
make install
noisd version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
noisd config node tcp://localhost:${NOIS_PORT}57
noisd config chain-id nois-1
noisd config keyring-backend os
noisd init $MONIKER --chain-id nois-1

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Nois/genesis.json > $HOME/.noisd/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Nois/addrbook.json > $HOME/.noisd/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.noisd/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.05unois\"|" $HOME/.noisd/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.noisd/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.noisd/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.noisd/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.noisd/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.noisd/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.noisd/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${NOIS_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${NOIS_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${NOIS_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${NOIS_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${NOIS_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${NOIS_PORT}66\"%" $HOME/.noisd/config/config.toml
sed -i -e "s%:1317%:${NOIS_PORT}17%g; s%:8080%:${NOIS_PORT}80%g; s%:9090%:${NOIS_PORT}90%g; s%:9091%:${NOIS_PORT}91%g; s%:8545%:${NOIS_PORT}45%g; s%:8546%:${NOIS_PORT}46%g; s%:6065%:${NOIS_PORT}65%g" $HOME/.noisd/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/noisd.service > /dev/null <<EOF
[Unit]
Description=Nois Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which noisd) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable noisd
```

## Start service and check the logs

```bash
sudo systemctl start noisd && sudo journalctl -u noisd -f --no-hostname -o cat
```

## Create wallet

```bash
noisd keys add wallet
```

## Check wallet balance

```bash
noisd q bank balances $(noisd keys show wallet -a)
```

## Create validator

```bash
noisd tx staking create-validator \
  --amount 1000000unois \
  --pubkey $(noisd tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id nois-1 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.05unois \
  -y
```

## Delete node

```bash
sudo systemctl stop noisd
sudo systemctl disable noisd
sudo rm -rf /etc/systemd/system/noisd.service
sudo systemctl daemon-reload
sudo rm -f $(which noisd) 
sudo rm -rf $HOME/.noisd
sudo rm -rf $HOME/noisd
```

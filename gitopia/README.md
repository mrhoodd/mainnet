# Install Guide Gitopia

## [Website](https://gitopia.com/) | [Discord](https://discord.com/invite/aqsKW3hUHD) | [Twitter](https://twitter.com/gitopiaDAO) | :satellite:[Explorer](https://explorer.moonbridge.team/gitopia)

**Chain ID:** gitopia | **Latest Version:** v3.3.0 | **Custom Port:** 154

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
GITOPIA_PORT=154
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
git clone https://github.com/gitopia/gitopia.git
cd gitopia
git checkout v3.3.0
make install
gitopiad version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
gitopiad config node tcp://localhost:${GITOPIA_PORT}57
gitopiad config chain-id gitopia
gitopiad config keyring-backend file
gitopiad init $MONIKER --chain-id gitopia

# Download genesis and addrbook
curl -Ls https://github.com/gitopia/mainnet/raw/master/genesis.tar.gz > $HOME/genesis.tar.gz
tar -xzf $HOME/genesis.tar.gz
mv genesis.json $HOME/.gitopia/config/genesis.json
rm $HOME/genesis.tar.gz
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Gitopia/addrbook.json > $HOME/.gitopia/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.gitopia/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.001ulore\"|" $HOME/.gitopia/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.gitopia/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.gitopia/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.gitopia/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.gitopia/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.gitopia/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.gitopia/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${GITOPIA_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${GITOPIA_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${GITOPIA_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${GITOPIA_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${GITOPIA_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${GITOPIA_PORT}66\"%" $HOME/.gitopia/config/config.toml
sed -i -e "s%:1317%:${GITOPIA_PORT}17%g; s%:8080%:${GITOPIA_PORT}80%g; s%:9090%:${GITOPIA_PORT}90%g; s%:9091%:${GITOPIA_PORT}91%g; s%:8545%:${GITOPIA_PORT}45%g; s%:8546%:${GITOPIA_PORT}46%g; s%:6065%:${GITOPIA_PORT}65%g" $HOME/.gitopia/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/gitopiad.service > /dev/null <<EOF
[Unit]
Description=Gitopia Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which gitopiad) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable gitopiad
```

## Start service and check the logs

```bash
sudo systemctl start gitopiad && sudo journalctl -u gitopiad -f --no-hostname -o cat
```

## Create wallet

```bash
gitopiad keys add wallet
```

## Check wallet balance

```bash
gitopiad q bank balances $(gitopiad keys show wallet -a)
```

## Create validator

```bash
gitopiad tx staking create-validator \
  --amount 1000000ulava \
  --pubkey $(gitopiad tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id gitopia \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.001ulore \
  -y
```

## Delete node

```bash
sudo systemctl stop gitopiad
sudo systemctl disable gitopiad
sudo rm -rf /etc/systemd/system/gitopiad.service
sudo systemctl daemon-reload
sudo rm -f $(which gitopiad) 
sudo rm -rf $HOME/.gitopia
sudo rm -rf $HOME/gitopia
```

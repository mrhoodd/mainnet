# Install Guide Source

## [Website](https://www.sourceprotocol.io/) | [Twitter](https://twitter.com/sourceprotocol_) | [Discord](https://discord.gg/zj8xxUCeZQ) | :satellite:[Explorer](https://explorer.moonbridge.team/source)

## Public endpoints

- API: <https://source.api.moonbridge.team>
- RPC: <https://source.rpc.moonbridge.team>

**Chain ID:** source-1 | **Latest Version:** v3.0.0 | **Custom Port:** 155

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
SOURCE_PORT=155
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
git clone https://github.com/Source-Protocol-Cosmos/source.git
cd ~/source
git fetch
git checkout v3.0.0
make install
sourced version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
sourced config node tcp://localhost:${SOURCE_PORT}57
sourced config chain-id source-1
sourced config keyring-backend file
sourced init $MONIKER --chain-id source-1

# Download genesis and addrbook
curl -Ls https://snapshots.moonbridge.team/mainnet/source/genesis.json > $HOME/.source/config/genesis.json
curl -Ls https://snapshots.moonbridge.team/mainnet/source/addrbook.json > $HOME/.source/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS="96d63849a529a15f037a28c276ea6e3ac2449695@34.222.1.252:26656,0107ac60e43f3b3d395fea706cb54877a3241d21@35.87.85.162:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.source/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.25usource\"|" $HOME/.source/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.source/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.source/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.source/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.source/config/app.toml

# Disable indexer
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.source/config/config.toml

# Enable Prometheus
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.source/config/config.toml

# Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${SOURCE_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${SOURCE_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${SOURCE_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${SOURCE_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${SOURCE_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${SOURCE_PORT}66\"%" $HOME/.source/config/config.toml
sed -i -e "s%:1317%:${SOURCE_PORT}17%g; s%:8080%:${SOURCE_PORT}80%g; s%:9090%:${SOURCE_PORT}90%g; s%:9091%:${SOURCE_PORT}91%g; s%:8545%:${SOURCE_PORT}45%g; s%:8546%:${SOURCE_PORT}46%g; s%:6065%:${SOURCE_PORT}65%g" $HOME/.source/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/sourced.service > /dev/null <<EOF
[Unit]
Description=Source Node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.source
ExecStart=$(which sourced) start --home $HOME/.source
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable sourced
```

## Start service and check the logs

```bash
sudo systemctl start sourced && sudo journalctl -u sourced -f --no-hostname -o cat
```

## Create wallet

```bash
sourced keys add wallet
```

## Recover wallet

```bash
sourced keys add wallet --recover
```

## Check wallet balance

```bash
sourced q bank balances $(sourced keys show wallet -a)
```

## Create validator

```bash
sourced tx staking create-validator \
  --amount 1000000usource \
  --pubkey $(sourced tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id source-1 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --fees=50000usource \
  -y
```

## Delete node

```bash
sudo systemctl stop sourced
sudo systemctl disable sourced
sudo rm -rf /etc/systemd/system/sourced.service
sudo systemctl daemon-reload
sudo rm -f $(which sourced) 
sudo rm -rf $HOME/.source
sudo rm -rf $HOME/source
```

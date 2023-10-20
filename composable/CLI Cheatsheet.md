
## Key management

Add new key

```bash
centaurid keys add wallet
```

Recover existing key

```bash
centaurid keys add wallet --recover
```

List all keys

```bash
centaurid keys list
```

Delete key

```bash
centaurid keys delete wallet
```

Export key to the file

```bash
centaurid keys export wallet
```

Import key from the file

```bash
centaurid keys import wallet wallet.backup
```

Wallet balance

```bash
centaurid q bank balances $(centaurid keys show wallet -a)
```

## Token management

Withdraw rewards from all validators

```bash
centaurid tx distribution withdraw-all-rewards --from wallet --chain-id centauri-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ppica -y
```

Withdraw rewards and commissions from your validator

```bash
centaurid tx distribution withdraw-rewards $(centaurid keys show wallet --bech val -a) --commission --from wallet --chain-id centauri-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ppica -y
```

Delegate tokens to yourself

```bash
centaurid tx staking delegate $(centaurid keys show wallet --bech val -a) 1000000ppica --from wallet --chain-id centauri-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ppica -y
```

Delegate tokens to validator

```bash
centaurid tx staking delegate <TO_VALOPER_ADDRESS> 1000000ppica --from wallet --chain-id centauri-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ppica -y
```

Redelegate tokens to another validator

```bash
centaurid tx staking redelegate $(centaurid keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 1000000ppica --from wallet --chain-id centauri-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ppica -y
```

Unbond tokens from your validator

```bash
centaurid tx staking unbond $(centaurid keys show wallet --bech val -a) 1000000ppica --from wallet --chain-id centauri-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ppica -y
```

Send tokens to the wallet

```bash
centaurid tx bank send wallet <TO_WALLET_ADDRESS> 1000000ppica --from wallet --chain-id centauri-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ppica -y
```

## Validator management

Validator info

```bash
centaurid status 2>&1 | jq .ValidatorInfo
```

Validator details

```bash
centaurid q staking validator $(centaurid keys show wallet --bech val -a)
```

Check if validator key is correct

```bash
[[ $(centaurid q staking validator $(centaurid keys show wallet --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(centaurid status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

List all active validators

```bash
centaurid q staking validators -oj --limit=1000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List all inactive validators

```bash
centaurid q staking validators -oj --limit=1000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Edit existing validator

```bash
centaurid tx staking edit-validator \
  --new-moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id centauri-1 \
  --commission-rate 0.10 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0ppica \
  -y
```

Jail reason

```bash
centaurid query slashing signing-info $(centaurid tendermint show-validator)
```

Unjail validator

```bash
centaurid tx slashing unjail --from wallet --chain-id centauri-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ppica -y
```

## Governance

Create a new offer

```bash
centaurid tx gov submit-proposal \
  --title "" \
  --description "" \
  --deposit 500000000000000000ppica \
  --type Text
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0ppica \
  -y 
```

List all proposals

```bash
centaurid query gov proposals
```

View proposal by ID

```bash
centaurid query gov proposal 1
```

Vote "YES"

```bash
centaurid tx gov vote 1 yes --from wallet --chain-id centauri-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ppica -y
```

Vote "NO"

```bash
centaurid tx gov vote 1 no --from wallet --chain-id centauri-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ppica -y
```

Vote "ABSTAIN"

```bash
centaurid tx gov vote 1 abstain --from wallet --chain-id centauri-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ppica -y
```

Vote "NOWITHVETO"

```bash
centaurid tx gov vote 1 NoWithVeto --from wallet --chain-id centauri-1 --gas-adjustment 1.4 --gas auto --gas-prices 0ppica -y
```

## Maintenance

Get sync info

```bash
centaurid status 2>&1 | jq .SyncInfo
```

Get node peer

```bash
echo $(centaurid tendermint show-node-id)'@'$(wget -qO- eth0.me)':'$(cat $HOME/.banksy/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

Get live peers

```bash
curl -sS http://localhost:15057/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

Enable Prometheus

```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.banksy/config/config.toml
```

Set minimum gas price

```bash
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0ppica\"|" $HOME/.banksy/config/app.toml
```

Disable indexer

```bash
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.banksy/config/config.toml
```

Enable indexer

```bash
sed -i -e 's|^indexer *=.*|indexer = "kv"|' $HOME/.banksy/config/config.toml
```

Update pruning

```bash
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.banksy/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.banksy/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.banksy/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.banksy/config/app.toml
```

Update ports

```bash
CUSTOM_PORT=150
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.banksy/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}17\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}91\"%" $HOME/.banksy/config/app.toml
```

Reset chain data

```bash
centaurid tendermint unsafe-reset-all --keep-addr-book --home $HOME/.banksy
```

Delete node

```bash
sudo systemctl stop centaurid
sudo systemctl disable centaurid
sudo rm -rf /etc/systemd/system/centaurid.service
sudo systemctl daemon-reload
sudo rm -f $(which centaurid) 
sudo rm -rf $HOME/.banksy
sudo rm -rf $HOME/composable-centauri
```

## Service Management

Status service

```bash
sudo systemctl status centaurid
```

Start service

```bash
sudo systemctl start centaurid
```

Stop service

```bash
sudo systemctl stop centaurid
```

Restart service

```bash
sudo systemctl restart centaurid
```

Logs service

```bash
sudo journalctl -u centaurid -f --no-hostname -o cat
```

Reload service

```bash
sudo systemctl daemon-reload
```

Enable service

```bash
sudo systemctl enable centaurid
```

Disable service

```bash
sudo systemctl disable centaurid
```

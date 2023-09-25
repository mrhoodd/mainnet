# Install Guide Bifrost

## [Website](https://bifrost.finance/) | [Discord](https://discord.com/invite/8DRBw2h5X4) | [Twitter](https://twitter.com/BifrostFinance) | :satellite:[Explorer](https://telemetry.polkadot.io/#list/0x9f28c6a68e0fc9646eff64935684f6eeeece527e37bbe1f213d22caa1d9d6bed)

## Download the binary file

```bash
wget https://github.com/bifrost-finance/bifrost/releases/download/bifrost-v0.9.64/bifrost
```

## Create a service account to run

```bash
adduser bifrost_service --system --no-create-home
```

## Create a directory to store the binary and data

```bash
mkdir /var/lib/bifrost-data
```

## Move the binary file to the created folder

```bash
mv ./bifrost /var/lib/bifrost-data
```

## Set ownership and permissions for the local directory where the chain data is stored

```bash
sudo chown -R bifrost_service /var/lib/bifrost-data
```

## Create configuration file

```bash
nano /etc/systemd/system/bifrost.service
```

:red_circle:Make the necessary corrections in the configuration file

```bash
[Unit]
After=network.target
StartLimitIntervalSec=0

[Service]
Restart=on-failure
RestartSec=10
User= root
SyslogIdentifier=bifrost
SyslogFacility=local7
KillSignal=SIGHUP
ExecStart=/var/lib/bifrost-data/bifrost \
        --name YOUR NAME \
        --collator \
        --force-authoring \
        --base-path /var/lib/bifrost-data\
        --ws-port=9944 \
        --port=30333 \
        --prometheus-external \
        --state-cache-size 0 \
        -- \
        --execution wasm

[Install]
WantedBy=multi-user.target
```

## Navigate to the directory where the binary file is located and grant access rights

```bash
cd /var/lib/bifrost-data
chmod +x bifrost
chown bifrost_service bifrost
cd $HOME
```

## Start the service

```bash
systemctl enable bifrost.service
systemctl start bifrost.service
systemctl status bifrost.service
```

## Check the logs

```bash
journalctl -u bifrost.service -f -o cat
```

## Delete node

```bash
sudo systemctl stop bifrost.service
sudo rm /etc/systemd/system/bifrost.service
sudo systemctl disable bifrost.service
rm -rf /var/lib/bifrost-data
```

# Install Guide Moonriver

## [Website](https://moonbeam.network/) | [Discord](https://discord.gg/moonbeam) | [Twitter](https://twitter.com/moonbeamnetwork) | :satellite:[Explorer](https://telemetry.polkadot.io/#list/0x401a1f9dca3da46f5c4091016c8a2f26dcea05865116b286f60f668207d1474b)

## Download the binary file

```bash
wget https://github.com/PureStake/moonbeam/releases/download/v0.33.0/moonbeam
```

## Create a service account to run

```bash
adduser moonriver_service --system --no-create-home
```

## Create a directory to store the binary and data

```bash
mkdir /var/lib/moonriver-data
```

## Move the binary file to the created folder

```bash
mv ./moonbeam /var/lib/moonriver-data
```

## Set ownership and permissions for the local directory where the chain data is stored

```bash
sudo chown -R moonriver_service /var/lib/moonriver-data
```

## Create configuration file

```bash
nano /etc/systemd/system/moonriver.service
```

>:red_circle:If you are setting up a collator node, make sure to follow the code snippets for Collator. Note that you have to:
>
>- Replace YOUR-NODE-NAME in two different places
>- Replace <50% RAM in MB> for 50% of the actual RAM your server has. For example, for 32 GB RAM, the value must be set to 16000. The minimum value is 2000, but it is below the recommended specs
>- Double-check that the binary is in the proper path as described below (ExecStart)
>- Double-check the base path if you've used a different directory

```bash
[Unit]
Description="Moonriver systemd service"
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=on-failure
RestartSec=10
User=moonriver_service
SyslogIdentifier=moonriver
SyslogFacility=local7
KillSignal=SIGHUP
ExecStart=/var/lib/moonriver-data/moonbeam \
     --validator \
     --port 30333 \
     --rpc-port 9933 \
     --ws-port 9944 \
     --execution wasm \
     --wasm-execution compiled \
     --trie-cache-size 0 \
     --db-cache <50% RAM in MB> \
     --base-path /var/lib/moonriver-data \
     --chain moonriver \
     --name "YOUR-NODE-NAME" \
     -- \
     --port 30334 \
     --rpc-port 9934 \
     --ws-port 9945 \
     --execution wasm \
     --name="YOUR-NODE-NAME (Embedded Relay)"

[Install]
WantedBy=multi-user.target
```

## Navigate to the directory where the binary file is located and grant access rights

```bash
cd /var/lib/moonriver-data/moonbeam
chmod +x moonbeam
chown moonriver_service moonbeam
cd $HOME
```

## Start the service

```bash
systemctl enable moonriver.service
systemctl start moonriver.service
systemctl status moonriver.service
```

## Check the logs

```bash
journalctl -u moonriver.service -f -o cat
```

## Delete node

```bash
sudo systemctl stop moonriver.service
sudo rm /etc/systemd/system/moonriver.service
sudo systemctl disable moonriver.service
rm -rf /var/lib/moonriver-data
```

# Upgrade

> If you want to update your client, you can keep the existing chain data intact and update only the binary file

## Stop the service

```bash
sudo systemctl stop moonriver.service
```

## Remove the old binary file

```bash
rm  /var/lib/moonriver-data/moonbeam
```

> Get the latest version of Moonbeam from the [Moonbeam GitHub Release page](https://github.com/PureStake/moonbeam/releases)

## Download the latest version of the binary file

```bash
wget https://github.com/PureStake/moonbeam/releases/download/<NEW VERSION TAG HERE>/moonbeam
```

## Move the binary to the data directory

```bash
mv ./moonbeam /var/lib/moonriver-data
```

## Navigate to the directory where the binary file is located and grant permissions

```bash
cd /var/lib/moonriver-data/moonbeam
chmod +x moonbeam
chown moonriver_service moonbeam
cd $HOME
```

## Register and start the service

```bash
systemctl enable moonriver.service 
systemctl start moonriver.service 
systemctl status moonriver.service
```

## Check the logs

```bash
journalctl -u moonriver.service -f -o cat
```

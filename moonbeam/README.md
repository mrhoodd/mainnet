# Install Guide Moonbeam

## [Website](https://moonbeam.network/) | [Discord](https://discord.gg/moonbeam) | [Twitter](https://twitter.com/moonbeamnetwork) | :satellite:[Explorer](https://telemetry.polkadot.io/#list/0xfe58ea77779b7abda7da4ec526d14db9b1e9cd40a217c34892af80a9b332b76d)

## Download the binary file

```bash
wget https://github.com/PureStake/moonbeam/releases/download/v0.33.0/moonbeam
```

## Create a service account to run

```bash
adduser moonbeam_service --system --no-create-home
```

## Create a directory to store the binary and data

```bash
mkdir /var/lib/moonbeam-data
```

## Move the binary file to the created folder

```bash
mv ./moonbeam /var/lib/moonbeam-data
```

## Set ownership and permissions for the local directory where the chain data is stored

```bash
sudo chown -R moonbeam_service /var/lib/moonbeam-data
```

## Create configuration file

```bash
nano /etc/systemd/system/moonbeam.service
```

>:red_circle:If you are setting up a collator node, make sure to follow the code snippets for Collator. Note that you have to:
>
>- Replace YOUR-NODE-NAME in two different places
>- Replace <50% RAM in MB> for 50% of the actual RAM your server has. For example, for 32 GB RAM, the value must be set to 16000. The minimum value is 2000, but it is below the recommended specs
>- Double-check that the binary is in the proper path as described below (ExecStart)
>- Double-check the base path if you've used a different directory

```bash
[Unit]
Description="Moonbeam systemd service"
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=on-failure
RestartSec=10
User=moonbeam_service
SyslogIdentifier=moonbeam
SyslogFacility=local7
KillSignal=SIGHUP
ExecStart=/var/lib/moonbeam-data/moonbeam \
     --validator \
     --port 30333 \
     --rpc-port 9933 \
     --ws-port 9944 \
     --execution wasm \
     --wasm-execution compiled \
     --trie-cache-size 0 \
     --db-cache <50% RAM in MB> \
     --base-path /var/lib/moonbeam-data \
     --chain moonbeam \
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
cd /var/lib/moonbeam-data/moonbeam
chmod +x moonbeam
chown moonbeam_service moonbeam
cd $HOME
```

## Start the service

```bash
systemctl enable moonbeam.service
systemctl start moonbeam.service
systemctl status moonbeam.service
```

## Check the logs

```bash
journalctl -u moonbeam.service -f -o cat
```

## Delete node

```bash
sudo systemctl stop moonbeam.service
sudo rm /etc/systemd/system/moonbeam.service
sudo systemctl disable moonbeam.service
rm -rf /var/lib/moonbeam-data
```

# Upgrade

> If you want to update your client, you can keep the existing chain data intact and update only the binary file

## Stop the service

```bash
sudo systemctl stop moonbeam.service
```

## Remove the old binary file

```bash
rm  /var/lib/moonbeam-data/moonbeam
```

> Get the latest version of Moonbeam from the [Moonbeam GitHub Release page](https://github.com/PureStake/moonbeam/releases)

## Download the latest version of the binary file

```bash
wget https://github.com/PureStake/moonbeam/releases/download/<NEW VERSION TAG HERE>/moonbeam
```

## Move the binary to the data directory

```bash
mv ./moonbeam /var/lib/moonbeam-data
```

## Navigate to the directory where the binary file is located and grant permissions

```bash
cd /var/lib/moonbeam-data/moonbeam
chmod +x moonbeam
chown moonbeam_service moonbeam
cd $HOME
```

## Register and start the service

```bash
systemctl enable moonbeam.service 
systemctl start moonbeam.service 
systemctl status moonbeam.service
```

## Check the logs

```bash
journalctl -u moonbeam.service -f -o cat
```


<p style="font-size:14px" align="right">
<a href="https://t.me/L0vd_staking" target="_blank">Join our telegram <img src="https://raw.githubusercontent.com/L0vd/screenshots/main/Telegram_logo.png" width="30"/></a>
<a href="https://l0vd.com/" target="_blank">Visit our website <img src="https://raw.githubusercontent.com/L0vd/screenshots/main/L0vd.png" width="30"/></a>
</p>

# Manual node setup
If you want to setup fullnode manually follow the steps below

## Setting up vars
Here you have to put name of your moniker (validator) that will be visible in explorer
```
NODENAME=<YOUR_MONIKER_NAME_GOES_HERE>
```

Save and import variables into system
```
PORT=<CHOOSE AVAILABLE PORT> #default 26
WALLET=<YOUR_WALLET_NAME_GOES_HERE> 
CHAIN_ID=defund-private-3

echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
echo "export WALLET=$WALLET" >> $HOME/.bash_profile
echo "export CHAIN_ID=$CHAIN_ID" >> $HOME/.bash_profile
echo "export PORT=$PORT" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Update packages
```
sudo apt update && sudo apt upgrade -y
```

## Install dependencies
```
sudo apt install curl build-essential git wget jq make gcc tmux chrony -y
```

## Install go
```
if ! [ -x "$(command -v go)" ]; then
  ver="1.18.2"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

## Download and build binaries
```
git clone https://github.com/defund-labs/defund.git
cd defund
git checkout v0.1.0
make install
```

## Config app
```
defundd config chain-id $DEFUND_CHAIN_ID
defundd config keyring-backend test
defundd config node tcp://localhost:$PORT657
```

## Init app
```
defundd init $NODENAME --chain-id $DEFUND_CHAIN_ID
```

## Download genesis and addrbook
```
wget -O $HOME/.defund/config/defund-private-3-gensis.tar.gz "https://raw.githubusercontent.com/defund-labs/testnet/main/defund-private-3/"
cd $HOME/.defund/config
tar -xzvf defund-private-3-gensis.tar.gz
rm -rf defund-private-3-gensis.tar.gz
```

## Set seeds and peers
```
SEEDS="85279852bd306c385402185e0125dffeed36bf22@38.146.3.194:26656,09ce2d3fc0fdc9d1e879888e7d72ae0fefef6e3d@65.108.105.48:11256"
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = "$SEEDS"/; s/^persistent_peers *=.*/persistent_peers = "$PEERS"/" $HOME/.defund/config/config.toml
```

## Set custom ports
```
sed -i.bak -e "s%^proxy_app = "tcp://127.0.0.1:26658"%proxy_app = "tcp://127.0.0.1:$PORT658"%; s%^laddr = "tcp://127.0.0.1:26657"%laddr = "tcp://127.0.0.1:$PORT657"%; s%^pprof_laddr = "localhost:6060"%pprof_laddr = "localhost:$PORT060"%; s%^laddr = "tcp://0.0.0.0:26656"%laddr = "tcp://0.0.0.0:$PORT656"%; s%^prometheus_listen_addr = ":26660"%prometheus_listen_addr = ":$PORT660"%" $HOME/.defund/config/config.toml
sed -i.bak -e "s%^address = "tcp://0.0.0.0:1317"%address = "tcp://0.0.0.0:$PORT317"%; s%^address = ":8080"%address = ":$PORT080"%; s%^address = "0.0.0.0:9090"%address = "0.0.0.0:$PORT090"%; s%^address = "0.0.0.0:9091"%address = "0.0.0.0:$PORT091"%; s%^address = "0.0.0.0:8545"%address = "0.0.0.0:$PORT545"%; s%^ws-address = "0.0.0.0:8546"%ws-address = "0.0.0.0:$PORT546"%" $HOME/.defund/config/app.toml
```

## Config pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = "$pruning"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = "$pruning_keep_recent"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = "$pruning_keep_every"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = "$pruning_interval"/" $HOME/.defund/config/app.toml
```

## Set minimum gas price and timeout commit
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = "0ufetf"/" $HOME/.defund/config/app.toml
```

## Enable prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.defund/config/config.toml
```

## Reset chain data
```
defundd tendermint unsafe-reset-all --home $HOME/.defund
```

## Create service
```
sudo tee /etc/systemd/system/defundd.service > /dev/null <<EOF
[Unit]
Description=fetf
After=network-online.target

[Service]
User=$USER
ExecStart=$(which defundd) start --home $HOME/.defund
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Register and start service
```
sudo systemctl daemon-reload
sudo systemctl enable defundd
sudo systemctl restart defundd && sudo journalctl -u defundd -f -o cat
```


#!/bin/bash
echo "=================================================="
echo "Skynodes"
echo "www.skynodejs.net"
echo "www.github.com/0xMrmoon"
echo "=================================================="


sleep 2

# Variables by Skynodes
DWS_WALLET=wallet
DWS=dewebd
DWS_ID=jagrat
DWS_PORT=12
DWS_FOLDER=.dws-node
DWS_FOLDER2=dws-node
DWS_VER=
DWS_REPO=https://github.com/deweb-services/deweb.git
DWS_GENESIS=https://raw.githubusercontent.com/deweb-services/deweb/main/genesis.json
DWS_ADDRBOOK=
DWS_MIN_GAS=0
DWS_DENOM=udws
DWS_SEEDS=


sleep 1

echo "export DWS_WALLET=${DWS_WALLET}" >> $HOME/.bash_profile
echo "export DWS=${DWS}" >> $HOME/.bash_profile
echo "export DWS_ID=${DWS_ID}" >> $HOME/.bash_profile
echo "export DWS_PORT=${DWS_PORT}" >> $HOME/.bash_profile
echo "export DWS_FOLDER=${DWS_FOLDER}" >> $HOME/.bash_profile
echo "export DWS_FOLDER2=${DWS_FOLDER2}" >> $HOME/.bash_profile
echo "export DWS_VER=${DWS_VER}" >> $HOME/.bash_profile
echo "export DWS_REPO=${DWS_REPO}" >> $HOME/.bash_profile
echo "export DWS_GENESIS=${DWS_GENESIS}" >> $HOME/.bash_profile
echo "export DWS_PEERS=${DWS_PEERS}" >> $HOME/.bash_profile
echo "export DWS_SEED=${DWS_SEED}" >> $HOME/.bash_profile
echo "export DWS_MIN_GAS=${DWS_MIN_GAS}" >> $HOME/.bash_profile
echo "export DWS_DENOM=${DWS_DENOM}" >> $HOME/.bash_profile
source $HOME/.bash_profile

sleep 1

if [ ! $DWS_NODENAME ]; then
	read -p "NODE: " DWS_NODENAME
	echo 'export HS_NODENAME='$DWS_NODENAME >> $HOME/.bash_profile
fi

echo -e "NODE: \e[1m\e[32m$DWS_NODENAME\e[0m"
echo -e "WALLET: \e[1m\e[32m$DWS_WALLET\e[0m"
echo -e "CHAIN: \e[1m\e[32m$DWS_ID\e[0m"
echo -e "PORT: \e[1m\e[32m$DWS_PORT\e[0m"
echo '================================================='

sleep 2


# Skynodes
echo -e "\e[1m\e[32m1. Upgrades... \e[0m" && sleep 1
sudo apt update && sudo apt upgrade -y


# Install.Packages Skynodes
echo -e "\e[1m\e[32m2. Packages... \e[0m" && sleep 1
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y

# GO 
echo -e "\e[1m\e[32m1. GO ... \e[0m" && sleep 1
ver="1.18.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version

sleep 1

# Library
echo -e "\e[1m\e[32m1. REPO ... \e[0m" && sleep 1
cd $HOME
git clone $DWS_REPO
cd $DWS_FOLDER2
make install

sleep 1

# skynodes
echo -e "\e[1m\e[32m1. ... \e[0m" && sleep 1
$DWS config chain-id $DWS_ID
$DWS config keyring-backend file
$DWS init $HS_NODENAME --chain-id $DWS_ID


# skynodes
wget $DWS_GENESIS -O $HOME/$HS_FOLDER/config/genesis.json
wget $DWS_ADDRBOOK -O $HOME/$HS_FOLDER/config/addrbook.json

# skynodes
SEEDS="$DWS_SEEDS"
PEERS="$DWS_PEERS"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/$DWS_FOLDER/config/config.toml

sleep 1


# config pruning
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/$DWS_FOLDER/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/$DWS_FOLDER/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/$DWS_FOLDER/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/$DWS_FOLDER/config/app.toml


# PORTS
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${DWS_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${DWS_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${DWS_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${DWS_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${DWS_PORT}660\"%" $HOME/$DWS_FOLDER/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${DWS_PORT}317\"%; s%^address = \":8080\"%address = \":${DWS_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${DWS_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${DWS_PORT}091\"%" $HOME/$DWS_FOLDER/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${HS_PORT}657\"%" $HOME/$HS_FOLDER/config/client.toml

# PROMETHEUS 
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/$DWS_FOLDER/config/config.toml

# MINIMUM GAS 
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.00125$HS_DENOM\"/" $HOME/$DWS_FOLDER/config/app.toml

# INDEXER 
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/$DWS_FOLDER/config/config.toml

# RESET 
$DWS tendermint unsafe-reset-all --home $HOME/$HS_FOLDER

echo -e "\e[1m\e[32m4. ... \e[0m" && sleep 1
# create service
sudo tee /etc/systemd/system/$DWS.service > /dev/null <<EOF
[Unit]
Description=$DWS
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which $DWS) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF


# 
sudo systemctl daemon-reload
sudo systemctl enable $DWS
sudo systemctl restart $DWS

echo '=============== SKYNODES ==================='
echo -e 'LOGLARI KONTROL ET: \e[1m\e[32msudo journalctl -u dewebd -f --no-hostname -o cat\e[0m'


source $HOME/.bash_profile

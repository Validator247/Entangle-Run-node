# Manual Installation

Update system and install build tools

    sudo apt update
    sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y

Install Go

    rm -rf $HOME/go
    sudo rm -rf /usr/local/go
    cd $HOME
    curl https://dl.google.com/go/go1.20.5.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
    cat <<'EOF' >>$HOME/.profile
    export GOROOT=/usr/local/go
    export GOPATH=$HOME/go
    export GO111MODULE=on
    export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
    EOF
    source $HOME/.profile
    go version

 Install Node

     cd $HOME
     rm -rf entangle-blockchain
     git clone https://github.com/Entangle-Protocol/entangle-blockchain.git
     cd entangle-blockchain
     git checkout ce539b81e760a3e75acd7fde9038c21fbe7b7baa
     make install
     entangled version

Initialize Node

    entangled init NodeName --chain-id=entangle_33133-1

Download Genesis

    curl -Ls https://ss-t.entangle.nodestake.top/genesis.json > $HOME/.entangled/config/genesis.json 

Download addrbook

    curl -Ls https://ss-t.entangle.nodestake.top/addrbook.json > $HOME/.entangled/config/addrbook.json

Create Service

    sudo tee /etc/systemd/system/entangled.service > /dev/null <<EOF
    [Unit]
    Description=entangled Daemon
    After=network-online.target
    [Service]
    User=$USER
    ExecStart=$(which entangled) start
    Restart=always
    RestartSec=3
    LimitNOFILE=65535
    [Install]
    WantedBy=multi-user.target
    EOF
    sudo systemctl daemon-reload
    sudo systemctl enable entangled

Download Snapshot(optional)

    SNAP_NAME=$(curl -s https://ss-t.entangle.nodestake.top/ | egrep -o ">20.*\.tar.lz4" | tr -d ">")
    curl -o - -L https://ss-t.entangle.nodestake.top/${SNAP_NAME}  | lz4 -c -d - | tar -x -C $HOME/.entangled

Launch Node

    sudo systemctl restart entangled
    journalctl -u entangled -f

# Create Validator
Add new key

    entangled keys add wallet

Export private key

    entangled keys unsafe-export-eth-key wallet

Recover existing key

    entangled keys add wallet --recover

List All key

    entangled keys list

Query Wallet Balance

    entangled q bank balances $(entangled keys show wallet -a)

Create Validator

    entangled tx staking create-validator \
    --amount "1000000000000000000aNGL" \
    --pubkey $(entangled tendermint show-validator) \
    --moniker "MONIKER" \
    --chain-id entangle_33133-1 \
    --commission-rate "0.05" \
    --commission-max-rate "0.20" \
    --commission-max-change-rate "0.01" \
    --min-self-delegation "1" \
    --gas-prices "10aNGL" \
    --gas "500000" \
    --gas-adjustment "1.5" \
    --from wallet \
    -y

Edit validator

    entangled tx staking edit-validator \
    --website "your-website"
    --chain-id entangle_33133-1 \
    --from wallet \
    --gas-adjustment 1.5 \
    --gas auto \
    --gas-prices "10aNGL" \
    -y


 Unjailed

    entangled tx slashing unjail --from wallet --chain-id=entangle_33133-1 --gas-prices 10aNGL --gas-adjustment 1.5 --gas auto -y

 Edit Validator

         shentud tx staking edit-validator \
         --chain-id=shentu-2.2 \
         --commission-rate=0.04 \
         --from=name_wallet \
         --gas-prices=0.1uctk \
         --gas-adjustment=1.5 \
         --gas=auto \
         -y


                 

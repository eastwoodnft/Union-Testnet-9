# Union-Testnet-9
Detailed steps for how to create a validator using a VPS!

1. Create a project on Linode

2. Spin up a droplet, you'll need at least 64GB of RAM, 500GB + of storage, and 16+ vCPU

3. Login via the console

4. Update system and install necessary tools
```
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential
sudo apt -qy upgrade
```
5. Install GO
```
sudo rm -rf /usr/local/go #only needed if you have a previous version of GO installed
curl -Ls https://go.dev/dl/go1.23.5.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
```
6. Download Binaries
```
wget https://github.com/unionlabs/union/releases/download/uniond%2Fv0.25.0/uniond.x86_64-linux.tar.gz
mkdir -p $HOME/.union/cosmovisor/genesis/bin
sudo tar -xzf uniond.x86_64-linux.tar.gz
cp result/bin/uniond $HOME/.union/cosmovisor/genesis/bin/uniond
chmod +x $HOME/.union/cosmovisor/genesis/bin/uniond
rm uniond.x86_64-linux.tar.gz
```
7. Create application symlinks
```
ln -s $HOME/.union/cosmovisor/genesis $HOME/.union/cosmovisor/current -f
sudo ln -s $HOME/.union/cosmovisor/current/bin/uniond /usr/local/bin/uniond -f
```
8. Download and install Cosmovisor
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.6.0
```
9. Create service
```
sudo tee /etc/systemd/system/union-testnet.service > /dev/null << EOF
[Unit]
Description=union node system service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --home=$HOME/.union
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.union"
Environment="DAEMON_NAME=uniond"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.union/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```
10. Reload the daemon and start the service
```
sudo systemctl daemon-reload
sudo systemctl enable union-testnet.service
```
11. Configure the alias for convinience
```
touch .bash_aliases
nano .bash_aliases
```
Add the following and save in .bash_aliases
```
alias uniond='uniond --home=$HOME/.union/'
```
12. Set node configuration
```
uniond config set client chain-id union-testnet-9
uniond config set client keyring-backend test
uniond config set client node tcp://0.0.0.0:26657
```
13. Initialize the node
```
uniond init $MONIKER --chain-id union-testnet-9 --home=$HOME/.union
```
14. Download genesis and addrbook
```
curl -Ls https://snapshots.kjnodes.com/union-testnet/genesis.json > $HOME/.union/config/genesis.json
curl -Ls https://snapshots.kjnodes.com/union-testnet/addrbook.json > $HOME/.union/config/addrbook.json
```
15. Add seeds
```
sed -i -e "s|^seeds *=.*|seeds = \"c2bf0d5b2ad3a1df0f4e9cc32debffa239c0af90@testnet.seed.poisonphang.com:26656\"|" $HOME/.union/config/config.toml
```
16. Set minimum gas price
```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0muno\"|" $HOME/.union/config/app.toml
```
17. Set pruning
```
sed -i -e 's|^pruning *=.*|pruning = "default"|' $HOME/.union/config/app.toml
```
19. Download Snapshot
```
curl -L https://snapshots.kjnodes.com/union-testnet/snapshot_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.union
[[ -f $HOME/.union/data/upgrade-info.json ]] && cp $HOME/.union/data/upgrade-info.json $HOME/.union/cosmovisor/genesis/upgrade-info.json
```
20. Start the Node and check the logs
```
sudo systemctl start union-testnet.service && sudo journalctl -u union-testnet.service -f --no-hostname -o cat
```

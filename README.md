# Union-Testnet-9
Detailed steps for how to create a validator using Digital Ocean Droplets!

1. Create a project on Digital Ocean

2. Spin up a droplet, you'll need at least 8GB of RAM, 250GB + of storage, and 4+ vCPU

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
mkdir -p $HOME/.union/cosmovisor/genesis/bin
wget -O $HOME/.union/cosmovisor/genesis/bin/uniond https://github.com/unionlabs/union/releases/download/uniond%2Fv0.25.0/uniond.x86_64-linux.tar.gz
chmod +x $HOME/.union/cosmovisor/genesis/bin/uniond
```
7. Create application symlinks
```
ln -s $HOME/.union/cosmovisor/genesis $HOME/.union/cosmovisor/current -f
sudo ln -s $HOME/.union/cosmovisor/current/bin/uniond /usr/local/bin/uniond -f
```



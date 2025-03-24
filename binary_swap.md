# Clone project repository
cd $HOME
rm -rf uniond
wget https://github.com/unionlabs/union/releases/download/uniond%2Fv0.25.1-rc2/uniond-release-x86_64-linux
sudo tar -xzf uniond-release-x86_64-linux
cp result/bin/uniond $HOME/.union/cosmovisor/upgrades/bin/uniond
chmod +x $HOME/.union/cosmovisor/upgrades/bin/uniond
rm uniond.x86_64-linux.tar.gz
cd uniond

# Build binaries
make build

# Prepare binaries for Cosmovisor
mkdir -p $HOME/.union/cosmovisor/upgrades/<unionBinary>/bin
mv build/uniond $HOME/.union/cosmovisor/upgrades/<unionBinary>/bin/
rm -rf build

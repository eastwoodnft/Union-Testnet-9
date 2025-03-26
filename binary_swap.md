# Download the new Test Binary and put it in a test-patch folder
```
cd $HOME
mkdir ~/.union/cosmovisor/upgrades/test-patch/
wget https://github.com/unionlabs/union/releases/download/uniond%2Fv0.25.1-rc2/uniond-release-x86_64-linux
sudo tar -xzf uniond-release-x86_64-linux
cp result/bin/uniond $HOME/.union/cosmovisor/upgrades/test-patch/uniond
chmod +x $HOME/.union/cosmovisor/upgrades/test-patch/uniond
rm uniond.x86_64-linux.tar.gz
```

## Prepare binaries for Cosmovisor

Remove the old symlink
```
rm -f /root/.union/cosmovisor/current
```
Create the symlink for the test-patch
```
ln -sf /root/.union/cosmovisor/upgrades/test-patch /root/.union/cosmovisor/current
```
Expected output:
```
lrwxrwxrwx 1 root root ... /root/.union/cosmovisor/current -> /root/.union/cosmovisor/upgrades/test-patch
```

## Verify the setup
Check symlink
```
ls -l /root/.union/cosmovisor/current
```
Expected output:
```
-r-xr-xr-x 1 root root 107796368 ... uniond
```
Check the bin/ directory:
```
ls -l /root/.union/cosmovisor/current/bin/
```
Expected output:
```
-r-xr-xr-x 1 root root 107796368 ... uniond
```
Test Binary Path
```
realpath /root/.union/cosmovisor/current/bin/uniond
```
Expected output:
```
/root/.union/cosmovisor/upgrades/test-patch/bin/uniond
```

## Start Cosmovisor with the new patch and verify
Start Cosmovisor
```
cosmovisor run start --home /root/.union
```
Confirm it uses the correct path:
```
systemctl status union.testnet-service
```
Expected output:
```
CGroup: /system.slice/union-testnet.service
    ├─262816 /root/go/bin/cosmovisor run start --home=/root/.union
    └─262837 /root/.union/cosmovisor/upgrades/test-patch/bin/uniond start --home=/root/.union
```

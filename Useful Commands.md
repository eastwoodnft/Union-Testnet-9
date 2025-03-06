

delegate
uniond tx staking delegate $(uniond keys show wallet --bech val -a) 1000000muno --from wallet --chain-id union-testnet-9



# Useful Commands for Union Blockchain

This document provides a collection of useful commands for managing keys, validators, tokens, governance, and node maintenance on the Union blockchain (testnet: `union-testnet-9`).

---

## 🔑 Key Management

### Add New Key
```
uniond keys add wallet
```
### Recover Existing Key
* Use more secure backend for mainnet *
```
uniond keys add wallet --recover --keyring-backend test
```
### List All Keys
```
uniond keys list
```
### Delete Key
```
uniond keys delete wallet
```
### Export Key to a File
```
uniond keys export wallet
```
### Import Key from a File
```
uniond keys import wallet wallet.backup
```
### Query Wallet Balance
```
uniond q bank balances $(uniond keys show wallet -a)
```
---

## 👷 Validator Management

*Note: Replace `YOUR_MONIKER_NAME`, `YOUR_KEYBASE_ID`, `YOUR_WEBSITE_URL`, `YOUR_SECURITY_EMAIL`, and `YOUR_DETAILS` with your own values.*

### Create New Validator
```
uniond tx staking create-validator <(cat <<EOF
{
  "pubkey": $(uniond comet show-validator),
  "amount": "1000000muno",
  "moniker": "YOUR_MONIKER_NAME",
  "identity": "YOUR_KEYBASE_ID",
  "website": "YOUR_WEBSITE_URL",
  "security": "YOUR_SECURITY_EMAIL",
  "details": "YOUR_DETAILS",
  "commission-rate": "0.05",
  "commission-max-rate": "0.20",
  "commission-max-change-rate": "0.05",
  "min-self-delegation": "1"
}
EOF
) --chain-id union-testnet-9 --from wallet
```

### Edit Existing Validator
```
uniond tx staking edit-validator --new-moniker "YOUR_MONIKER_NAME" --identity "YOUR_KEYBASE_ID" --details "YOUR_DETAILS" --website "YOUR_WEBSITE_URL" --chain-id union-testnet-9 --commission-rate 0.05 --from wallet
```
### Unjail Validator
```
uniond tx slashing unjail --from wallet --chain-id union-testnet-9
```
### Check Jail Reason
```
uniond query slashing signing-info $(uniond comet show-validator)
```
### List All Active Validators
```
uniond q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
### List All Inactive Validators
```
uniond q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
### View Validator Details
```
uniond q staking validator $(uniond keys show wallet --bech val -a)
```
---

## 💲 Token Management

### Withdraw Rewards from All Validators
```
uniond tx distribution withdraw-all-rewards --from wallet --chain-id union-testnet-9
```
### Withdraw Commission and Rewards from Your Validator
```
uniond tx distribution withdraw-rewards $(uniond keys show wallet --bech val -a) --commission --from wallet --chain-id union-testnet-9
```
### Delegate Tokens to Yourself
```
uniond tx staking delegate $(uniond keys show wallet --bech val -a) 1000000muno --from wallet --chain-id union-testnet-9
```
### Delegate Tokens to Another Validator
```
uniond tx staking delegate <TO_VALOPER_ADDRESS> 1000000muno --from wallet --chain-id union-testnet-9
```
### Redelegate Tokens to Another Validator
```
uniond tx staking redelegate $(uniond keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 1000000muno --from wallet
```
### Unbond Tokens from Your Validator
```
uniond tx staking unbond $(uniond keys show wallet --bech val -a) 1000000muno --from wallet --chain-id union-testnet-9
```
### Send Tokens to a Wallet
```
uniond tx bank send wallet <TO_WALLET_ADDRESS> 1000000muno --from wallet --chain-id union-testnet-9
```
### Claim Validator Rewards/Commission
```
uniond tx distribution withdraw-validator-commission wallet --from wallet
```
---

## 🗳 Governance

### List All Proposals
```
uniond query gov proposals
```
### View Proposal by ID
```
uniond query gov proposal 1
```
### Vote ‘Yes’
```
uniond tx gov vote 1 yes --from wallet --chain-id union-testnet-9
```
### Vote ‘No’
```
uniond tx gov vote 1 no --from wallet --chain-id union-testnet-9
```
### Vote ‘Abstain’
```
uniond tx gov vote 1 abstain --from wallet --chain-id union-testnet-9
```
### Vote ‘NoWithVeto’
```
uniond tx gov vote 1 NoWithVeto --from wallet --chain-id union-testnet-9
```
---

## ⚡️ Utility

### Update Ports
```
CUSTOM_PORT=110
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.union/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}17\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}91\"%" $HOME/.union/config/app.toml
```
### Update Indexer

#### Disable Indexer
```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.union/config/config.toml
```
#### Enable Indexer
```
sed -i -e 's|^indexer *=.*|indexer = "kv"|' $HOME/.union/config/config.toml
```
### Update Pruning
```
sed -i -e 's|^pruning *=.*|pruning = "custom"|' -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' -e 's|^pruning-interval *=.*|pruning-interval = "19"|' $HOME/.union/config/app.toml
```
---

## 🚨 Maintenance

### Get Validator Info
```
uniond status 2>&1 | jq .validator_info
```
### Get Sync Info
```
uniond status 2>&1 | jq .sync_info
```
### Get Node Peer
```
echo $(uniond comet show-node-id)'@'$(curl -4s ifconfig.me)':'$(cat $HOME/.union/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```
### Check if Validator Key Is Correct
```
[[ $(uniond q staking validator $(uniond keys show wallet --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(uniond status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```
### Get Live Peers
```
curl -sS http://localhost:26657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```
### Set Minimum Gas Price
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0muno\"/" $HOME/.union/config/app.toml
```
### Enable Prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.union/config/config.toml
```
### Reset Chain Data
```
uniond comet unsafe-reset-all --keep-addr-book --home $HOME/.union --keep-addr-book
```
### Remove Node
*Warning: This deletes all chain data! Backup `priv_validator_key.json` before proceeding!*
```
cd $HOME
sudo systemctl stop union-testnet.service
sudo systemctl disable union-testnet.service
sudo rm /etc/systemd/system/union-testnet.service
sudo systemctl daemon-reload
rm -f $(which uniond)
rm -rf $HOME/.union
```
---

## ⚙️ Service Management

### Reload Service Configuration
```
sudo systemctl daemon-reload
```
### Enable Service
```
sudo systemctl enable union-testnet.service
```
### Disable Service
```
sudo systemctl disable union-testnet.service
```
### Start Service
```
sudo systemctl start union-testnet.service
```
### Stop Service
```
sudo systemctl stop union-testnet.service
```
### Restart Service
```
sudo systemctl restart union-testnet.service
```
### Check Service Status
```
sudo systemctl status union-testnet.service
```
### Check Service Logs
```
sudo journalctl -u union-testnet.service -f --no-hostname -o cat
```
### Save Log File
```
sudo journalctl -u union-testnet.service -S <date in YYYY-MM-DD HH:MM:SS> --no-tail > <log_file_name>.log
```
---

This file is tailored for the `union-testnet-9` chain. Adjust commands (e.g., `chain-id`, token amounts, or ports) as needed for your specific setup.

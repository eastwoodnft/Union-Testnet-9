# Install & Configure Prometheus
Prometheus is a monitoring system that collects metrics via HTTP endpoints.

## Install Prometheus (Ubuntu/Debian)
```
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus /var/lib/prometheus
```

## Download Prometheus:
```
cd /tmp
curl -LO https://github.com/prometheus/prometheus/releases/download/v3.2.0-rc.1/prometheus-3.2.0-rc.1.linux-amd64.tar.gz
tar xvf prometheus-3.2.0-rc.1.linux-amd64.tar.gz
sudo mv prometheus-3.2.0-rc.1.linux-amd64.tar.gz/prometheus /usr/local/bin/
sudo mv prometheus-3.2.0-rc.1.linux-amd64.tar.gz/promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool
```

## Create Prometheus Configuration File
```
sudo nano /etc/prometheus/prometheus.yml
```

## Paste the following:
```
global:
  scrape_interval: 15s  # Adjust based on your needs

scrape_configs:
  - job_name: 'cosmos-validator'
    static_configs:
      - targets: ['localhost:26660']  # Adjust to your node’s Prometheus metrics port
```

## Create Prometheus Systemd Service
```
sudo nano /etc/systemd/system/prometheus.service
```

## Paste:
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus

[Install]
WantedBy=multi-user.target
```
## Start & Enable Prometheus
```
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
```

## Verify:
```
sudo systemctl status prometheus
```
Prometheus is now running at:
➡ http://localhost:9090

# Enable Prometheus Metrics on Your Cosmos Validator
Cosmos nodes expose metrics at port 26660. Enable it in config.toml:
```
nano ~/.gaia/config/config.toml
```
Find [instrumentation] and update:
```
[instrumentation]
prometheus = true
prometheus_listen_addr = ":26660"
```
Restart your node:
```
sudo systemctl restart gaiad
```
Now, Prometheus can scrape your validator's metrics from:
```
http://localhost:26660/metrics
```

# Install & Configure Grafana
Grafana provides a visual dashboard for Prometheus metrics.

## Install Grafana
```
sudo apt install -y apt-transport-https software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt update
sudo apt install grafana -y
```

## Start & Enable Grafana
```
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```
Grafana is now running at:
```
http://localhost:3000
```
(Default login: admin / admin)
If you don't have a local web browser and want to monitor remotely, continue below.

# Connect Grafana to Prometheus
1. Open Grafana (http://your-server-ip:3000).
2. Go to Settings → Data Sources.
3. Click Add data source → Choose Prometheus.
4. Set the URL to:
```
http://localhost:9090
```
Click Save & Test.

# Import a Cosmos Validator Dashboard
Instead of creating a dashboard from scratch, import a prebuilt Cosmos Validator dashboard:

Go to Grafana → Dashboards.\
Click New → Import.\
Use [Union_Validator.json](https://github.com/eastwoodnft/Union-Testnet-9/blob/main/Union_Validator.json)\
Click Load.\
Select Prometheus as the data source.\
Click Import.\
🎉 Now you have a real-time Cosmos validator monitoring dashboard!\

✅ Final Setup Checklist\
✔ Prometheus running on localhost:9090\
✔ Cosmos node exposing metrics on localhost:26660\
✔ Grafana running on localhost:3000\
✔ Imported the Cosmos validator dashboard\

If you can't access Prometheus (http://your-server-ip:9090) or Grafana (http://your-server-ip:3000), here are the steps to troubleshoot and fix it.

Check if Prometheus & Grafana are Running
Run these commands:

Check Prometheus
```
sudo systemctl status prometheus
```
If it's not running, restart it:
```
sudo systemctl restart prometheus
```
If it fails, check logs:
```
sudo journalctl -u prometheus --no-pager --lines=50
```
Check Grafana
```
sudo systemctl status grafana-server
```
If it’s not running, restart:
```
sudo systemctl restart grafana-server
```
If it fails, check logs:
```
sudo journalctl -u grafana-server --no-pager --lines=50
```

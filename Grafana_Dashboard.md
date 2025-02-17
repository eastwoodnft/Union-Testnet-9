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
Enter Dashboard ID: 14574 (Cosmos Validator Metrics).\
Click Load.\
Select Prometheus as the data source.\
Click Import.\
🎉 Now you have a real-time Cosmos validator monitoring dashboard!\

✅ Final Setup Checklist\
✔ Prometheus running on localhost:9090\
✔ Cosmos node exposing metrics on localhost:26660\
✔ Grafana running on localhost:3000\
✔ Imported the Cosmos validator dashboard\

# If you can't access Prometheus (http://your-server-ip:9090) or Grafana (http://your-server-ip:3000), here are the steps to troubleshoot and fix it.

1. Check if Prometheus & Grafana are Running
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
2. Check Firewall Rules
Your firewall might be blocking access to ports 9090 (Prometheus) and 3000 (Grafana).

List Firewall Rules
sh
Copy
Edit
sudo ufw status
Allow Prometheus & Grafana
sh
Copy
Edit
sudo ufw allow 9090/tcp
sudo ufw allow 3000/tcp
sudo ufw reload
For CentOS (firewalld):

sh
Copy
Edit
sudo firewall-cmd --permanent --add-port=9090/tcp
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload
3. Check If Prometheus & Grafana Are Listening on the Right IP
By default, Prometheus & Grafana might bind only to localhost (127.0.0.1), which means they aren't accessible remotely.

Modify Prometheus Config
Edit:

sh
Copy
Edit
sudo nano /etc/systemd/system/prometheus.service
Find this line:

ini
Copy
Edit
ExecStart=/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus
Change to:

ini
Copy
Edit
ExecStart=/usr/local/bin/prometheus --web.listen-address=0.0.0.0:9090 --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus
Save & restart Prometheus:

sh
Copy
Edit
sudo systemctl daemon-reload
sudo systemctl restart prometheus
Modify Grafana Config
Edit:

sh
Copy
Edit
sudo nano /etc/grafana/grafana.ini
Find:

ini
Copy
Edit
;http_addr =
;http_port = 3000
Change to:

ini
Copy
Edit
http_addr = 0.0.0.0
http_port = 3000
Save & restart Grafana:

sh
Copy
Edit
sudo systemctl restart grafana-server
4. Verify That Ports Are Open
Check if Prometheus & Grafana are listening:

sh
Copy
Edit
sudo netstat -tulnp | grep -E "9090|3000"
Expected output:

nginx
Copy
Edit
tcp        0      0 0.0.0.0:9090       0.0.0.0:*       LISTEN      <PID>/prometheus
tcp        0      0 0.0.0.0:3000       0.0.0.0:*       LISTEN      <PID>/grafana
If not, restart the services.

5. Try Accessing via IP Instead of Domain
Instead of:

arduino
Copy
Edit
http://your-domain.com:9090
http://your-domain.com:3000
Try:

arduino
Copy
Edit
http://your-server-ip:9090
http://your-server-ip:3000
You can check your server IP with:

sh
Copy
Edit
ip a | grep inet
6. If Running on a Cloud Server, Open Ports in Provider's Firewall
Some cloud providers (AWS, GCP, Azure) block ports by default.

For AWS (EC2)
Go to AWS Console → EC2.
Select your instance.
Navigate to Security Groups.
Add an Inbound Rule:
Protocol: TCP
Port: 9090, 3000
Source: Your IP or 0.0.0.0/0 (for public access).
For Google Cloud
sh
Copy
Edit
gcloud compute firewall-rules create allow-prometheus-grafana --allow tcp:9090,tcp:3000 --source-ranges=0.0.0.0/0
7. Try Restarting the Server
If nothing works, try a full restart:

sh
Copy
Edit
sudo reboot
✅ Final Fix Checklist
✔ Prometheus & Grafana services running (systemctl status)
✔ Firewall allows ports 9090 & 3000 (ufw status)
✔ Services listening on 0.0.0.0 (netstat -tulnp)
✔ Cloud provider firewall rules allow traffic
✔ Access via http://your-server-ip:9090 instead of localhost

Try these fixes and let me know if you're still having issues! 🚀

## Prometheus Deployment on Rocky Linux 9

### Step 1: System Preparation

```bash
sudo dnf update -y
sudo useradd --no-create-home --shell /bin/false prometheus
```
> ⚠️ Note: Prometheus runs as a dedicated system user. Skipping this may lead to permission or security issues.

### Step 2: Download and Extract Prometheus

Visit https://prometheus.io/download/ and choose the latest version, then:
```bash
cd /tmp
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
tar -xzf prometheus-2.52.0.linux-amd64.tar.gz
cd prometheus-2.52.0.linux-amd64
```
> ⚠️ Note: Replace the version 2.52.0 with the latest stable version as needed.

### Step 3: Move Binaries and Set Permissions

```bash
sudo mv prometheus /usr/local/bin/
sudo mv promtool /usr/local/bin/
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/
sudo chown -R prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool \
  /etc/prometheus /var/lib/prometheus
```
> ⚠️ Note: Permissions are critical. Incorrect ownership can prevent Prometheus from starting.

### Step 4: Create Prometheus Systemd Service

```bash
sudo tee /etc/systemd/system/prometheus.service > /dev/null <<EOF
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
EOF
```
> ⚠️ Note: Ensure the paths match your Prometheus binary and config locations.

### Step 6: Configure Firewall

```bash
sudo firewall-cmd --permanent --add-port=9090/tcp
sudo firewall-cmd --reload
```
> ⚠️ Note: Port 9090 is the default Prometheus web interface. Ensure it is open to access the dashboard.

### Step 7: Access Prometheus Web Interface

Open your browser and navigate to:
```bash
http://<your_server_ip>:9090
```
> You should see the Prometheus dashboard and be able to query metrics using the built-in PromQL interface.

### Step 8: Basic Configuration Overview

Edit /etc/prometheus/prometheus.yml to define targets and scrape intervals:
```bash
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```
> ⚠️ Note: Any syntax error in the config file will prevent Prometheus from restarting. Use promtool check config /etc/prometheus/prometheus.yml to validate.

### Final Notes

- Prometheus binary: /usr/local/bin/prometheus
- Config file: /etc/prometheus/prometheus.yml
- Console templates: /etc/prometheus/consoles/
- Data storage: /var/lib/prometheus/
- Logs: Use journalctl -u prometheus
Optional enhancements:
- Add exporters (e.g., Node Exporter) for system metrics
- Alertmanager setup for email/SMS alerts
- Use Grafana to visualize metrics from Prometheus

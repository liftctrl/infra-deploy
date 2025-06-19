## Grafana Deployment on Rocky Linux 9

### Step 1: System Preparation

```bash
sudo dnf update -y
```
> ⚠️ Note: Ensure your system is up-to-date before installing external repositories.

### Step 2: Add Grafana Repository

```bash
sudo tee /etc/yum.repos.d/grafana.repo > /dev/null <<EOF
[grafana]
name=Grafana OSS
baseurl=https://rpm.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
EOF
```
> ⚠️ Note: This configures the official Grafana OSS YUM repository. Ensure your system has internet access to reach it.

### Step 3: Install Grafana

```bash
sudo dnf install -y grafana
```
> ⚠️ Note: If the installation fails, verify the repo URL and GPG key accessibility.

### Step 4: Enable and Start Grafana Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now grafana-server
```
> ⚠️ Note: Use sudo systemctl status grafana-server to verify if Grafana started correctly.

### Step 5: Configure Firewall

```bash
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload
```
> ⚠️ Note: Grafana’s web UI runs on port 3000. Ensure this is allowed through your firewall.

### Step 6: Access Grafana Web Interface

Open your browser and navigate to:
```bash
http://<your_server_ip>:3000
```
Default credentials:
- Username: admin
- Password: admin
You'll be prompted to change the password after the first login.
> ⚠️ Note: Always change the default credentials to secure your instance.

### Step 7: Configure Grafana (Optional Basics)

1. After logging in:
   - Add a Data Source:
   - Go to Settings > Data Sources.
   - Choose Prometheus, MySQL, or another data source.
   - Fill in the details (e.g., Prometheus URL: http://localhost:9090) and click Save & Test.
2. Import a Dashboard:
   - Go to + > Import.
   - Enter a dashboard ID from https://grafana.com/grafana/dashboards or upload a JSON file.
> ⚠️ Note: Dashboards and data sources are not persistent unless saved explicitly. Consider exporting configurations for backup.

### Step 8: Enable Grafana at Boot

```bash
sudo systemctl enable grafana-server
```
> ⚠️ Note: This ensures Grafana automatically starts on system reboot.

### Final Notes

- Grafana service: grafana-server
- Config file: /etc/grafana/grafana.ini
- Logs: /var/log/grafana/grafana.log
- Web UI: http://<server_ip>:3000
To configure HTTPS:
Edit /etc/grafana/grafana.ini:
```bash
[server]
protocol = https
cert_file = /etc/grafana/certs/grafana.crt
cert_key = /etc/grafana/certs/grafana.key
```
Then restart:
```bash
sudo systemctl restart grafana-server
```
> ⚠️ Note: You must generate or obtain a valid SSL certificate before enabling HTTPS.

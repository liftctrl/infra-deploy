## Apache Tomcat Deployment on Rocky Linux 9

### Step 1: System Preparation

```bash
sudo dnf update -y
sudo dnf install -y java-17-openjdk-devel wget
```
> ⚠️ Note: Tomcat requires Java. Use a compatible version like OpenJDK 11 or 17. Verify with java -version.

### Step 2: Create Tomcat User

```bash
sudo useradd -r -m -U -d /opt/tomcat -s /bin/false tomcat
```
> ⚠️ Note: Tomcat should run as a non-privileged user for security.

### Step 3: Download and Extract Tomcat

```bash
cd /tmp
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.24/bin/apache-tomcat-10.1.24.tar.gz
tar -xzvf apache-tomcat-10.1.24.tar.gz
sudo mv apache-tomcat-10.1.24 /opt/tomcat/latest
sudo ln -s /opt/tomcat/latest /opt/tomcat/latest
```
> ⚠️ Note: Always use the latest stable version from https://tomcat.apache.org. Adjust paths accordingly.

### Step 4: Set Permissions

```bash
sudo chown -R tomcat: /opt/tomcat
sudo chmod +x /opt/tomcat/latest/bin/*.sh
```
> ⚠️ Note: Tomcat scripts must be executable by the Tomcat user.

### Step 5: Create Systemd Service

```bash
sudo tee /etc/systemd/system/tomcat.service > /dev/null <<EOF
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/java-17-openjdk"
Environment="CATALINA_PID=/opt/tomcat/latest/temp/tomcat.pid"
Environment="CATALINA_HOME=/opt/tomcat/latest"
Environment="CATALINA_BASE=/opt/tomcat/latest"

ExecStart=/opt/tomcat/latest/bin/startup.sh
ExecStop=/opt/tomcat/latest/bin/shutdown.sh

Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```
> ⚠️ Note: Ensure JAVA_HOME matches your installed Java version. Use readlink -f $(which java) to confirm.

### Step 6: Reload Daemon and Start Tomcat

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now tomcat
```
> ⚠️ Note: Check Tomcat status with sudo systemctl status tomcat. Logs are in /opt/tomcat/latest/logs/.

### Step 7: Configure Firewall

```bash
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```
> ⚠️ Note: Tomcat listens on port 8080 by default. Ensure it's open if accessing from outside.

### Step 8: Access Tomcat Web Interface

Open your browser and go to:
```bash
http://<your_server_ip>:8080
```
You should see the Tomcat welcome page.
> ⚠️ Note: No authentication is required by default. To enable admin access, proceed to the next step.

### Step 9: Enable Admin Interface (Optional)

Edit the tomcat-users.xml file:
```bash
sudo nano /opt/tomcat/latest/conf/tomcat-users.xml
```
Add inside <tomcat-users>:
```bash
<role rolename="manager-gui"/>
<role rolename="admin-gui"/>
<user username="admin" password="admin_password" roles="manager-gui,admin-gui"/>
```
> ⚠️ Note: Replace the username and password securely. Restart Tomcat to apply.
```bash
sudo systemctl restart tomcat
```

### Final Notes

- Tomcat Home: /opt/tomcat/latest
- Service: tomcat
- Logs: /opt/tomcat/latest/logs
- Web UI: http://<server_ip>:8080
- Default Ports:
  - HTTP: 8080
  - HTTPS: 8443 (requires SSL setup)
Optional Enhancements:
- Configure SSL/TLS in server.xml
- Deploy .war apps in /opt/tomcat/latest/webapps/
- Use Nginx reverse proxy with SSL

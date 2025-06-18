## Jenkins Deployment on Rocky Linux 9

### Step 1: System Preparation

```bash
sudo dnf update -y
sudo dnf install -y java-17-openjdk
```
> ⚠️ Note: Jenkins requires Java. Java 17 is recommended. If not installed, Jenkins will fail to start.

### Step 2: Add Jenkins Repository

```bash
sudo curl --silent --location https://pkg.jenkins.io/redhat-stable/jenkins.repo | sudo tee /etc/yum.repos.d/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
```
> ⚠️ Note: If the GPG key changes, check the official Jenkins RPM repository for the updated key.

### Step 3: Install Jenkins

```bash
sudo dnf install -y jenkins
```
> ⚠️ Note: If the package is not found, ensure the repo file and key were added correctly in step 2.

### Step 4: Start and Enable Jenkins Service

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```
> ⚠️ Note: If Jenkins fails to start, check with sudo systemctl status jenkins and verify Java is installed correctly.

### Step 5: Adjust Firewall to Allow Web Access

```bash
sudo firewall-cmd --permanent --zone=public --add-port=8080/tcp
sudo firewall-cmd --reload
```
> ⚠️ Note: Jenkins by default runs on port 8080. Skipping this will block web access to the Jenkins UI.

### Step 6: Retrieve Initial Admin Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
> ⚠️ Note: This password is required during the first-time web setup. Store it securely.

### Step 7: Access Jenkins Web Interface

Open your browser and go to:
```bash
http://<your_server_ip>:8080
```
> ⚠️ Note: Use the initial admin password from step 6 when prompted. If the page doesn't load, check firewall rules and service status.

### Step 8: Install Suggested Plugins

Follow the on-screen instructions after logging in:
- Choose Install Suggested Plugins (recommended)
- Create a new admin user
- Configure Jenkins instance URL (optional)
> ⚠️ Note: Plugin installation may take a few minutes. Ensure the server has a stable internet connection.

### Step 9: Enable Jenkins to Start at Boot (Optional)

```bash
sudo systemctl enable jenkins
```
> ⚠️ Note: Jenkins will automatically start on system boot after enabling this.

### Final Notes

- Jenkins data directory: /var/lib/jenkins
- Jenkins configuration file: /etc/sysconfig/jenkins
- Logs are stored at: /var/log/jenkins/jenkins.log
- To change the default port (8080), edit:
```bash
sudo nano /etc/sysconfig/jenkins
```
Look for:
```bash
JENKINS_PORT="8080"
```
- For Git integration, install Git:
```bash
sudo dnf install -y git
```
- For production setups:
  - Use reverse proxy (e.g., NGINX)
  - Enable HTTPS
  - Backup /var/lib/jenkins regularly

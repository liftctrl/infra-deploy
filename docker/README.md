## Docker Deployment on Rocky Linux 9

### Step 1: System Preparation

```bash
sudo dnf update -y
sudo dnf install -y yum-utils device-mapper-persistent-data lvm2
```
> ⚠️ Note: Missing dependencies like lvm2 or device-mapper will cause Docker installation or volume management to fail.

### Step 2: Add Docker Repository

```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
> ⚠️ Note: Even on Rocky Linux, Docker uses the CentOS repo. This is safe and officially recommended by Docker.

### Step 3: Install Docker Engine

```bash
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
> ⚠️ Note: Use --nobest if package version conflicts occur. Example: sudo dnf install -y docker-ce --nobest.

### Step 4: Start and Enable Docker Service

```bash
sudo systemctl start docker
sudo systemctl enable docker
```
> ⚠️ Note: If Docker fails to start, check sudo systemctl status docker and verify kernel version and storage drivers.

### Step 5: Test Docker Installation

```bash
sudo docker run hello-world
```
> ⚠️ Note: This downloads a test image from Docker Hub. Ensure your system has internet access.

### Step 6: Manage Docker as a Non-root User (Optional)

```bash
sudo usermod -aG docker $USER
newgrp docker
```
> ⚠️ Note: The new group takes effect after re-logging or running newgrp. Skipping this step may result in permission denied errors for non-root users.

### Step 7: Enable Firewall Access (For Docker Services)

If your containers expose ports (e.g., web services):
```bash
sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
sudo firewall-cmd --permanent --zone=public --add-port=443/tcp
sudo firewall-cmd --reload
```
> ⚠️ Note: Docker does not automatically manage firewall rules. You must allow necessary ports manually.

### Step 8: Enable Docker to Start at Boot (Optional)

```bash
sudo systemctl enable docker
```
> ⚠️ Note: This ensures Docker and its containers start after a system reboot.

### Final Notes

- Use docker ps, docker images, and docker volume ls for environment inspection.
- Docker logs are accessible via sudo journalctl -u docker.
- To install Docker Compose manually:
```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose version
```
- SELinux in enforcing mode may cause networking or volume access issues in containers. You can:
  - Add :z or :Z to volume mounts
  - Or temporarily set SELinux to permissive for debugging:
  ```bash
  sudo setenforce 0
  ```

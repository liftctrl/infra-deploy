## Redis Deployment on Rocky Linux 9

### Step 1: System Preparation

```bash
sudo dnf update -y
```
> ⚠️ Note: It's always good to update the system first to avoid any issues with outdated dependencies or package versions.

### Step 2: Install Redis

```bash
sudo dnf install -y redis
```
> ⚠️ Note: If the redis package is not found, ensure that the EPEL repository is enabled (sudo dnf install epel-release).

### Step 3: Start and Enable Redis Service

```bash
sudo systemctl start redis
sudo systemctl enable redis
```
> ⚠️ Note: If Redis fails to start, check the service status using sudo systemctl status redis or check the logs (sudo journalctl -xe).

### Step 4: Verify Redis Installation

```bash
redis-cli ping
```
You should get:
```bash
PONG
```
> ⚠️ Note: If you don’t get PONG, Redis might not be running correctly. Ensure the service is active with sudo systemctl status redis.

### Step 5: Adjust Firewall (Optional)

If you need to expose Redis to remote clients (default port is 6379):
```bash
sudo firewall-cmd --permanent --zone=public --add-port=6379/tcp
sudo firewall-cmd --reload
```
> ⚠️ Note: Make sure Redis is not exposed publicly for production environments. Bind it to localhost or restrict access with firewalls.

### Step 6: Configure Redis (Optional)

Edit the Redis configuration file for custom settings:
```bash
sudo nano /etc/redis.conf
```
Key settings you might modify:
- bind: Specify which IP addresses Redis listens on (e.g., bind 127.0.0.1 for localhost-only).
- protected-mode: If yes, Redis will only accept connections from clients on the same machine by default.
- requirepass: Set a password for Redis connections.
```bash
requirepass my_secure_password
```
After editing, restart Redis:
```bash
sudo systemctl restart redis
```
> ⚠️ Note: If you change requirepass, make sure to authenticate with the password in the redis-cli or any client.

### Step 7: Test Redis with Password (if set)

```bash
redis-cli -a my_secure_password ping
```
> ⚠️ Note: If you enabled requirepass, ensure you provide the correct password when connecting.

### Step 8: Enable Redis to Start at Boot (Optional)

```bash
sudo systemctl enable redis
```
> ⚠️ Note: This ensures Redis starts automatically on system boot.

### Final Notes

- Redis logs are located at /var/log/redis/redis.log. Use tail -f /var/log/redis/redis.log to monitor.
- Redis data is stored by default in /var/lib/redis/. Make sure this directory has proper permissions.
- Consider setting up Redis for persistence by adjusting appendonly and save settings in /etc/redis.conf for AOF or RDB persistence.
- For production, ensure Redis is secured behind a firewall or within a private network, and consider using TLS encryption for connections.

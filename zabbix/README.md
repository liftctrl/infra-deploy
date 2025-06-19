## Zabbix Deployment on Rocky Linux 9

### Step 1: System Preparation

```bash
sudo dnf update -y
sudo dnf install -y epel-release
```
> ⚠️ Note: EPEL is required for some Zabbix dependencies. Missing this may cause installation failure.

### Step 2: Install Zabbix Repository

```bash
sudo dnf install -y https://repo.zabbix.com/zabbix/6.0/rhel/9/x86_64/zabbix-release-6.0-4.el9.noarch.rpm
sudo dnf clean all
```
> ⚠️ Note: Replace version 6.0 with the desired Zabbix version (e.g., 6.4 LTS) if needed.

### Step 3: Install Zabbix Server, Frontend, Agent, and MySQL

```bash
sudo dnf install -y zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-sql-scripts zabbix-agent mariadb-server
```
> ⚠️ Note: Make sure you install the database server (mariadb-server) if you don't have a database backend already.

### Step 4: Start and Configure MariaDB

```bash
sudo systemctl enable --now mariadb
sudo mysql_secure_installation
```
> ⚠️ Note: Set a strong root password. Say yes to removing anonymous users, disallowing remote root, and reloading privileges.

### Step 5: Create Zabbix Database and User

```bash
mysql -u root -p
```
Then run in the MySQL prompt:
```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER zabbix@localhost IDENTIFIED BY 'zabbix_password';
GRANT ALL PRIVILEGES ON zabbix.* TO zabbix@localhost;
FLUSH PRIVILEGES;
EXIT;
```
> ⚠️ Note: Replace 'zabbix_password' with a secure password and keep it for configuration.

### Step 6: Import Initial Schema and Data

```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -u zabbix -p zabbix
```
> ⚠️ Note: Use the Zabbix DB password created in the previous step. Importing can take several minutes.

### Step 7: Configure Zabbix Server Database Connection

Edit the Zabbix server config:
```bash
sudo nano /etc/zabbix/zabbix_server.conf
```
Set:
```bash
DBPassword=zabbix_password
```
> ⚠️ Note: Ensure this matches the DB password you created. 

### Step 8: Start and Enable Zabbix Services

```bash
sudo systemctl enable --now zabbix-server zabbix-agent httpd php-fpm
```
> ⚠️ Note: If services fail to start, check logs under /var/log/zabbix/ or use systemctl status.

### Step 9: Set PHP Timezone

Edit the PHP config for Zabbix:
```bash
sudo nano /etc/php-fpm.d/zabbix.conf
```
Uncomment and set:
```bash
php_value[date.timezone] = YOUR_TIMEZONE
```
Example:
```bash
php_value[date.timezone] = Asia/Kuala_Lumpur
```
Then restart services:
```bash
sudo systemctl restart php-fpm httpd
```
> ⚠️ Note: Without timezone set, the frontend may show errors or warnings.

### Step 10: Configure Firewall

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-port=10051/tcp
sudo firewall-cmd --permanent --add-port=10050/tcp
sudo firewall-cmd --reload
```
> ⚠️ Note: Ports 10050 (agent) and 10051 (server) are essential for communication.

### Step 11: Access Zabbix Web Interface

Open your browser and navigate to:
```bash
http://<your_server_ip>/zabbix
```
Initial login credentials:
- Username: Admin
- Password: zabbix
> ⚠️ Note: Change the default password immediately after first login.

### Final Notes

- Zabbix Server config: /etc/zabbix/zabbix_server.conf
- Zabbix Agent config: /etc/zabbix/zabbix_agentd.conf
- PHP settings: /etc/php-fpm.d/zabbix.conf
- Apache config: /etc/httpd/conf.d/zabbix.conf
- Log files: /var/log/zabbix/, /var/log/httpd/, /var/log/php-fpm/
For production:
- Enable HTTPS with a reverse proxy (NGINX or Apache with Let's Encrypt).
- Monitor Zabbix service status:
```bash
sudo systemctl status zabbix-server
sudo systemctl status zabbix-agent
```
- Consider enabling SELinux policies if you're using SELinux in enforcing mode.

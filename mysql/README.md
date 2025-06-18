## MySQL Deployment on Rocky Linux 9

### Step 1: System Preparation 

```bash
sudo dnf update -y
```
> ⚠️ Note: Outdated system packages may cause dependency issues or prevent the MySQL repo from being added properly.

### Step 2: Add MySQL Yum Repository

```bash
sudo dnf install -y https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf repolist enabled | grep mysql
```
> ⚠️ Note: If the repository URL changes or becomes unreachable, download it manually from the MySQL website.

### Step 3: Install MySQL Server

```bash
sudo dnf install -y mysql-community-server
```
> ⚠️ Note: If you receive GPG errors, ensure the GPG key was installed with the repo package, or use --nogpgcheck as a last resort.

### Step 4: Start and Enable MySQL Service

```bash
sudo systemctl start mysqld
sudo systemctl enable mysqld
```
> ⚠️ Note: If MySQL fails to start, check sudo systemctl status mysqld or sudo journalctl -xe for detailed errors.

### Step 5: Get Temporary Root Password

```bash
sudo grep 'temporary password' /var/log/mysqld.log
```
> ⚠️ Note: You must retrieve this password for the initial mysql_secure_installation. Don’t skip this step.

### Step 6: Run Security Configuration

```bash
sudo mysql_secure_installation
```
Follow the prompts:
- Enter temporary password
- Set a new root password
- Remove anonymous users
- Disallow root login remotely (recommended)
- Remove test database
- Reload privilege tables
> ⚠️ Note: If you input a weak password, the script may reject it depending on password policy.

### Step 7: Test MySQL Login

```bash
mysql -u root -p
```
> ⚠️ Note: Use the new password set in the previous step. If login fails, retry mysql_secure_installation or reset the root password.

### Step 8: Allow Remote Connections (Optional)

Edit bind-address in config:
```bash
sudo nano /etc/my.cnf
```
Change or add:
```bash
bind-address = 0.0.0.0
```
Restart MySQL:
```bash
sudo systemctl restart mysqld
```
Open firewall:
```bash
sudo firewall-cmd --permanent --add-service=mysql
sudo firewall-cmd --reload
```
> ⚠️ Note: Allowing 0.0.0.0 opens MySQL to all IPs — secure with user restrictions and firewall rules.

### Final Notes

- MySQL logs are located at /var/log/mysqld.log
- Use systemctl restart mysqld after any configuration change
- To create a new user:
```bash
CREATE USER 'user'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'user'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
- SELinux may block remote access or custom data directories — use semanage or set proper contexts with chcon

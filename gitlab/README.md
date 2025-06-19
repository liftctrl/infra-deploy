## GitLab Deployment on Rocky Linux 9

### Step 1: System Preparation

```bash
sudo dnf update -y
sudo dnf install -y curl policycoreutils openssh-server perl
sudo systemctl enable --now sshd
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```
> ⚠️ Note: GitLab requires sshd (for SSH), curl, and firewall access on HTTP/HTTPS. If these aren’t installed or configured correctly, web access and SSH cloning won’t work.

### Step 2: Install Postfix (For Email Notifications)

```bash
sudo dnf install -y postfix
sudo systemctl enable --now postfix
```
> ⚠️ Note: Postfix is used for sending email notifications. You can configure an external SMTP later if desired, but Postfix is needed for local email.

### Step 3: Add GitLab Repository

```bash
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
```
> curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash

### Step 4: Install GitLab Community Edition

Set the EXTERNAL_URL to your domain or IP address:
```bash
sudo EXTERNAL_URL="http://gitlab.example.com" dnf install -y gitlab-ce
```
> ⚠️ Note: Replace http://gitlab.example.com with your actual domain or server IP address. This is the URL GitLab will be accessible at.

### Step 5: Configure GitLab

If you need to change or confirm the external URL, modify the GitLab configuration file:
```bash
sudo nano /etc/gitlab/gitlab.rb
```
Change the external_url line to:
```bash
external_url "http://gitlab.example.com"
```
Once you modify this, reconfigure GitLab:
```bash
sudo gitlab-ctl reconfigure
```
> ⚠️ Note: Misconfigured URLs can break your GitLab access. Always remember to reconfigure after making changes.

### Step 6: Access GitLab Web Interface

Open your browser and navigate to:
```bash
http://<your_server_ip>
```
On the first login, use the following credentials:
- Username: root
- Password: Retrieve the initial root password using:
```bash
sudo cat /etc/gitlab/initial_root_password
```
> ⚠️ Note: The initial root password is auto-generated and valid only for the first login. Change it immediately after logging in.

### Step 7: Enable GitLab to Start at Boot (Optional)

```bash
sudo systemctl enable gitlab-runsvdir
```
> ⚠️ Note: This ensures GitLab’s services start automatically after reboot.

### Step 8: Configure Backups (Highly Recommended)

To configure automatic backups for your GitLab instance, modify the gitlab.rb file:
```bash
sudo nano /etc/gitlab/gitlab.rb
```
Set the backup path:
```bash
gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"
```
Then, manually create a backup:
```bash
sudo gitlab-backup create
```
> ⚠️ Note: Automate backups using cron jobs, and ensure there is enough disk space in your backup directory.

### Step 9: Managing GitLab Services

You can manage GitLab services as follows:
- Check status of all services:
```bash
sudo gitlab-ctl status
```
- Restart all GitLab services:
```bash
sudo gitlab-ctl restart
```
- View logs:
```bash
sudo gitlab-ctl tail
```
> ⚠️ Note: Always monitor GitLab services and logs to ensure it is running smoothly.

### Final Notes

- GitLab configuration file: /etc/gitlab/gitlab.rb
- GitLab logs are stored in: /var/log/gitlab/
- GitLab data is stored in: /var/opt/gitlab/
- GitLab backups are stored in: /var/opt/gitlab/backups/
To enable HTTPS (recommended for production), configure SSL either using GitLab’s built-in Let's Encrypt integration or set up a reverse proxy with NGINX/Apache.
> For HTTPS via Let’s Encrypt:
- Modify /etc/gitlab/gitlab.rb:
```bash
external_url "https://gitlab.example.com"
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/ssl/certs/gitlab.crt"
nginx['ssl_certificate_key'] = "/etc/ssl/private/gitlab.key"
```
- Run: sudo gitlab-ctl reconfigure

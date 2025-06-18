## NGINX Deployment on Rocky Linux 9

### Step 1: System Preparation

```bash
sudo dnf update -y
```
> ⚠️ Note: Skipping system updates may leave outdated packages that cause compatibility issues during or after installation.

### Step 2: Install NGINX

```bash
sudo dnf install -y nginx
```
> ⚠️ Note: If dnf cannot find the nginx package, ensure EPEL is enabled or run sudo dnf install epel-release first.

### Step 3: Start and Enable NGINX Service

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```
> ⚠️ Note: If the service doesn't start, check for port conflicts or SELinux-related denials (sudo journalctl -xe).

### Step 4: Adjust Firewall Settings

```bash
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```
> ⚠️ Note: Forgetting to open ports will block web access. Use firewall-cmd --list-all to confirm rules.

### Step 5: Confirm Installation

```bash
sudo systemctl status nginx
```
Then open your server’s IP or domain in a browser:
```bash
http://<your_server_ip>
```
> ⚠️ Note: If the page doesn't load, check that the server IP is correct, firewall allows access, and NGINX is running.

### Step 6: Configure NGINX Server Block (Optional)

Create a configuration file:
```bash
sudo nano /etc/nginx/conf.d/example.conf
```
Example config:
```bash
server {
    listen 80;
    server_name example.com;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
> ⚠️ Note: Make sure server_name matches your DNS entry and that /var/www/html exists with an index.html file.

### Step 7: Test and Reload Configuration

```bash
sudo nginx -t
sudo systemctl reload nginx
```
> ⚠️ Note: If nginx -t fails, fix syntax errors before reloading. Always test config before applying changes.

### Step 8: Create a Sample Web Page

```bash
echo "<h1>NGINX on Rocky Linux 9</h1>" | sudo tee /var/www/html/index.html
```
> ⚠️ Note: Ensure NGINX has permission to access the web root directory.

### Final Notes

- Logs are in /var/log/nginx/ – check error.log for debugging.
- Use sudo systemctl restart nginx after any major config changes.
- To serve PHP or proxy to apps, install extra modules or edit location blocks.
- SELinux may block custom web roots — use chcon -Rt httpd_sys_content_t /your/path to allow access.

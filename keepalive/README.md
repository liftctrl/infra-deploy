## Keepalived Deployment on Rocky Linux 9

### Step 1: System Preparation

```bash
sudo dnf update -y
sudo dnf install -y keepalived
```
> ⚠️ Note: Ensure both nodes (if using HA) are using the same version of Keepalived to avoid compatibility issues.

### Step 2: Enable and Start Keepalived

```bash
sudo systemctl enable --now keepalived
```
> ⚠️ Note: If Keepalived fails to start, verify the syntax of your configuration file (/etc/keepalived/keepalived.conf).

### Step 3: Configure Keepalived

Create or edit the configuration file:
```bash
sudo nano /etc/keepalived/keepalived.conf
```
Example for MASTER node:
```bash
vrrp_instance VI_1 {
    state MASTER
    interface ens160
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1234
    }
    virtual_ipaddress {
        192.168.1.100
    }
}
```
Example for BACKUP node:
```bash
vrrp_instance VI_1 {
    state BACKUP
    interface ens160
    virtual_router_id 51
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1234
    }
    virtual_ipaddress {
        192.168.1.100
    }
}
```
> ⚠️ Note: Replace ens160 with your actual network interface. The virtual_ipaddress must be the same on both nodes. Ensure auth_pass matches exactly.

### Step 4: Restart Keepalived After Configuration

```bash
sudo systemctl restart keepalived
```
> ⚠️ Note: After restarting, you can check if the virtual IP is assigned with ip addr show.

### Step 5: Configure Firewall (if needed)

Keepalived uses multicast by default:
```bash
sudo firewall-cmd --permanent --add-rich-rule='rule protocol value="vrrp" accept'
sudo firewall-cmd --reload
```
> ⚠️ Note: Some environments (e.g., cloud providers) may block multicast. Consider using unicast in those cases.

### Step 6: Verify Keepalived Status

```bash
sudo systemctl status keepalived
ip addr show | grep 192.168.1.100
```
> ⚠️ Note: The virtual IP should only appear on the current MASTER node.

### Step 7: Enable Logging (Optional)

To enable logging:
```bash
sudo nano /etc/sysconfig/keepalived
```
Add:
```bash
KEEPALIVED_OPTIONS="-D -d -S 0"
```
Then restart:
```bash
sudo systemctl restart keepalived
```
Logs will appear in:
```bash
sudo journalctl -u keepalived
```
> ⚠️ Note: Use -S 0 for logging to syslog. You can adjust log levels for more or less verbosity.

### Final Notes

- Config file: /etc/keepalived/keepalived.conf
- Logs: journalctl -u keepalived or /var/log/messages (if using syslog)
- Service: keepalived
- Use tcpdump -i <iface> vrrp to verify VRRP packets if troubleshooting
- If using unicast in restricted environments, update config:
```bash
unicast_peer {
    192.168.1.2
}
```
- Test failover: shut down or stop keepalived on the MASTER node and watch the BACKUP node take over the virtual IP.

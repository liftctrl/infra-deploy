## LVS (IPVS) Deployment on Rocky Linux 9

### Step 1: System Preparation

```bash
sudo dnf update -y
sudo dnf install -y ipvsadm
```
> ⚠️ Note: ipvsadm is the user-space tool for managing IPVS rules. Without it, LVS configuration is not possible.

### Step 2: Enable IPVS Kernel Module

```bash
sudo modprobe ip_vs
sudo modprobe ip_vs_rr
sudo modprobe ip_vs_wrr
sudo modprobe ip_vs_sh
```
> ⚠️ Note: Ensure these modules are loaded. You can add them to /etc/modules-load.d/ipvs.conf for persistence:
```bash
echo -e "ip_vs\nip_vs_rr\nip_vs_wrr\nip_vs_sh" | sudo tee /etc/modules-load.d/ipvs.conf
```

### Step 3: Enable IP Forwarding

```bash
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
> ⚠️ Note: IP forwarding is necessary for LVS to route traffic. LVS won’t function without this.

### Step 4: Configure LVS Virtual Service (NAT or DR mode)

Basic NAT mode example using ipvsadm:
```bash
sudo ipvsadm -A -t 192.168.1.100:80 -s rr
sudo ipvsadm -a -t 192.168.1.100:80 -r 192.168.1.201:80 -m
sudo ipvsadm -a -t 192.168.1.100:80 -r 192.168.1.202:80 -m
```
Explanation:
- 192.168.1.100: Virtual IP (VIP)
- -s rr: Round-robin scheduler
- -m: Masquerading (NAT mode)
- 192.168.1.201, 192.168.1.202: Real servers
> ⚠️ Note: Ensure LVS host can reach real servers and that return traffic is routed through LVS (important in NAT mode).

### Step 5: View and Verify LVS Configuration

```bash
sudo ipvsadm -L -n
```
> ⚠️ Note: Use -n to avoid DNS lookup. Check that all real servers are correctly listed under the VIP.

### Step 6: Persistent Configuration (Optional)

To persist rules, install and enable the ipvsadm service:
```bash
sudo systemctl enable --now ipvsadm
```
Save the current IPVS configuration:
```bash
sudo ipvsadm-save > /etc/sysconfig/ipvsadm
```
> ⚠️ Note: Without this step, your configuration will be lost after a reboot.

### Step 7: Configure Real Servers (For DR Mode Only)

If using Direct Routing (DR mode) instead of NAT:
- Assign the VIP (e.g., 192.168.1.100) to loopback interface on real servers:
```bash
sudo ip addr add 192.168.1.100/32 dev lo
```
- Disable ARP on loopback to prevent VIP conflict:
```bash
echo 1 | sudo tee /proc/sys/net/ipv4/conf/lo/arp_ignore
echo 2 | sudo tee /proc/sys/net/ipv4/conf/lo/arp_announce
```
Make persistent by adding to /etc/sysctl.conf:
```bash
net.ipv4.conf.lo.arp_ignore = 1
net.ipv4.conf.lo.arp_announce = 2
```
> ⚠️ Note: Without ARP suppression, multiple real servers may respond to ARP for VIP, breaking LVS-DR.

### Step 8: Firewall Configuration

On LVS Director (load balancer):
```bash
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload
```
> ⚠️ Note: Ensure the VIP port (e.g., 80, 443) is allowed.

### Final Notes

- Tool: ipvsadm (or nftables in advanced scenarios)
- Config file for persistence: /etc/sysconfig/ipvsadm
- Useful commands:
  - ipvsadm -L -n – List current LVS rules
  - ipvsadm-save – Save rules
  - ipvsadm-restore – Restore rules
- Consider using Keepalived for high availability (VIP failover) with LVS

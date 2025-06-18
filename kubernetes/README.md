## Kubernetes Deployment on Rocky Linux 9 (Using kubeadm)

### Step 1: System Preparation

```bash
sudo dnf update -y
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```
> ⚠️ Note: Kubernetes requires swap to be disabled and SELinux in permissive mode. If not done, kubelet will fail to start.

### Step 2: Configure Kernel Modules and Sysctl

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sudo modprobe br_netfilter
sudo sysctl --system
```
> ⚠️ Note: If these settings are skipped, Kubernetes networking (especially kube-proxy) will not function correctly.

### Step 3: Install Container Runtime (Containerd)

```bash
sudo dnf install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```
> ⚠️ Note: Docker is not recommended for new clusters. Use containerd unless there’s a specific reason to use Docker.

### Step 4: Add Kubernetes Repository

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key
EOF
```
> ⚠️ Note: Make sure to replace v1.30 with the latest stable version if needed. GPG key errors can be avoided by double-checking the key URL.

### Step 5: Install Kubernetes Tools

```bash
sudo dnf install -y kubelet kubeadm kubectl
sudo systemctl enable kubelet
```
> ⚠️ Note: Do not start kubelet yet — it will fail without initialization (kubeadm init).

### Step 6: Initialize Kubernetes Master Node

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```
> ⚠️ Note: Replace --pod-network-cidr depending on the network plugin you choose. This step outputs a kubeadm join command — save it!

### Step 7: Configure kubectl Access

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
> ⚠️ Note: If this is skipped, kubectl will not be able to connect to the cluster.

### Step 8: Install a Pod Network Add-on (e.g., Calico)

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
```
> ⚠️ Note: Pods won’t start until the network plugin is deployed. Ensure your CIDR matches what was set during kubeadm init.

### Step 9: Join Worker Nodes (From Each Worker Node)

Run the kubeadm join ... command provided by the master node:
```bash
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
> ⚠️ Note: Tokens expire after 24 hours. To regenerate, run:
```bash
kubeadm token create --print-join-command
```

### Step 10: Verify Cluster Status

```bash
kubectl get nodes
kubectl get pods --all-namespaces
```
> ⚠️ Note: Nodes will show NotReady until the pod network is fully deployed and functioning.

### Final Notes

- Use crictl to debug container runtimes: sudo crictl ps -a
- Check logs with: sudo journalctl -u kubelet -f
- For high availability or multi-master clusters, configure etcd and control plane replication.
- Be cautious with firewall rules:
```bash
sudo firewall-cmd --permanent --add-port=6443/tcp --add-port=10250/tcp --add-port=10251/tcp
sudo firewall-cmd --reload
```

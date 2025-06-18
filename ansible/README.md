## Ansible Deployment on Rocky Linux 9

### Step 1: System Preparation

```bash
sudo dnf update -y
sudo dnf install -y epel-release
```
> ⚠️ Note: Failing to install epel-release may cause package dependencies to be missing later.

### Step 2: Install Ansible

```bash
sudo dnf install -y ansible
```
> ⚠️ Note: The default version from Rocky 9’s repos might not be the latest. If you need the latest version, use pip or install from source (see step 6 optional).

### Step 3: Confirm Installation

```bash
ansible --version
```
> ⚠️ Note: If ansible command is not found, ensure /usr/bin is in your PATH or re-check installation status.

### Step 4: Configure Ansible Inventory

Create/edit the inventory file:
```bash
sudo nano /etc/ansible/hosts
```
Add target hosts:
```bash
[webservers]
192.168.1.10
192.168.1.11
```
> ⚠️ Note: Wrong host format or syntax will cause playbook execution to fail.

### Step 5: Setup SSH Access to Remote Hosts

```bash
ssh-keygen -t rsa
ssh-copy-id user@192.168.1.10
```
> ⚠️ Note: Ensure the remote host allows SSH and the user has correct permissions.

### Step 6 (Optional): Install Latest Ansible via pip

```bash
sudo dnf install -y python3-pip
pip3 install --user ansible
```
> ⚠️ Note: Installing via pip installs under ~/.local/bin, make sure it’s in your $PATH.
```bash
echo 'export PATH=$PATH:~/.local/bin' >> ~/.bashrc
source ~/.bashrc
```

### Step 7: Run a Test Command

```bash
ansible all -m ping
```
> ⚠️ Note: This will fail if:
- Inventory is incorrect
- SSH keys are missing or not working
- Remote user lacks permissions

### Step 8: Create and Run a Simple Playbook

Create a YAML playbook file:
```bash
nano test-playbook.yml
```
```bash
---
- hosts: all
  become: true
  tasks:
    - name: Ensure Apache is installed
      dnf:
        name: httpd
        state: present
```
Run it:
```bash
ansible-playbook test-playbook.yml
```
> ⚠️ Note: become: true requires the user to have sudo privileges on the target machines.

### Final Notes

- Always validate YAML with yamllint if errors occur.
- Use ansible-config dump to inspect current Ansible configuration.
- Use ansible-playbook -i inventory_file your_playbook.yml -vvv for verbose debugging.
- SELinux may block some operations — check audit.log if unexpected behavior happens.


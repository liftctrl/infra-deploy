## Terraform Deployment on Rocky Linux 9

### Step 1: System Preparation

```bash
sudo dnf update -y
sudo dnf install -y wget unzip
```
> ⚠️ Note: Terraform is not included in default Rocky Linux repositories. You need to install it manually from HashiCorp.

### Step 2: Add HashiCorp YUM Repository

```bash
sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
```
> ⚠️ Note: This repository includes Terraform and other HashiCorp tools like Vault, Packer, Consul, etc.

### Step 3: Install Terraform

```bash
sudo dnf install -y terraform
```
> ⚠️ Note: You can check the installed version with:
```bash
terraform -version
```

### Step 4: Verify Installation

```bash
terraform -help
```
> ⚠️ Note: If terraform is not found, check your $PATH or confirm successful installation.

### Step 5: Create Your First Terraform Project

```bash
mkdir ~/terraform-project
cd ~/terraform-project
```
Create a sample config file:
```bash
nano main.tf
```
Example content:
```bash
terraform {
  required_providers {
    null = {
      source = "hashicorp/null"
    }
  }
}

resource "null_resource" "example" {
  provisioner "local-exec" {
    command = "echo Hello, Terraform!"
  }
}
```
> ⚠️ Note: This is a basic example using the null provider for testing purposes.

### Step 6: Initialize Terraform

```bash
terraform init
```
> ⚠️ Note: This downloads necessary provider plugins and initializes the working directory.

### Step 7: Apply Terraform Plan

```bash
terraform plan
terraform apply -auto-approve
```
> ⚠️ Note: Always inspect the plan before applying in production. -auto-approve skips the confirmation prompt.

### Step 8: Clean Up Resources

```bash
terraform destroy -auto-approve
```
> ⚠️ Note: This safely removes all resources defined in the Terraform configuration.

### Optional: Install Specific Terraform Version

```bash
TF_VER=1.7.5
wget https://releases.hashicorp.com/terraform/${TF_VER}/terraform_${TF_VER}_linux_amd64.zip
unzip terraform_${TF_VER}_linux_amd64.zip
sudo mv terraform /usr/local/bin/
```
> ⚠️ Note: This is useful if you want to pin a specific version or use multiple versions across projects.

### Final Notes

- Binary location: /usr/bin/terraform or /usr/local/bin/terraform
- Config files: .tf files in working directory
- Logs: Terraform does not log to file by default; use TF_LOG for debug:
```bash
TF_LOG=DEBUG terraform apply
```
- Documentation: https://developer.hashicorp.com/terraform/docs
Common Errors:
- "provider not found": Run terraform init again.
- "command not found": Ensure /usr/bin or /usr/local/bin is in your $PATH.
- "403 Forbidden" when accessing repo: Verify internet access or use proxy if needed.

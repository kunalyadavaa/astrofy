---
title: "Terraform"
description: "Terraform is a powerful Infrastructure as Code (IaC) tool developed by HashiCorp that allows you to define and provision infrastructure across multiple providers.In this guide, we’ll configure **Terraform to automate VM provisioning in a Proxmox VE Cluster**. This enables reproducible, version-controlled deployments—perfect for home labs or scalable environments."
pubDate: "March 15 2025"
heroImage: "/Terraform icon.webp"
---

# ⚙️ Infrastructure as Code in Home Lab: Terraform Setup with Proxmox VE Cluster

## 🌱 Introduction

Terraform is a powerful Infrastructure as Code (IaC) tool developed by HashiCorp that allows you to define and provision infrastructure across multiple providers.

In this guide, we’ll configure Terraform to automate VM provisioning in a Proxmox VE Cluster. This enables reproducible, version-controlled deployments—perfect for home labs or scalable environments.

---

## 🧩 Prerequisites

| Component               | Requirement                         |
| ----------------------- | ----------------------------------- |
| Proxmox VE              | Cluster with at least 2 nodes       |
| Terraform               | Installed on local machine (latest) |
| Proxmox API Token       | With sufficient permissions         |
| Proxmox Provider Plugin | Installed in Terraform              |
| SSH Key Pair            | For VM access (optional)            |

> 📸 **Photo Suggestion 1**: A diagram showing Terraform on a workstation managing a 3-node Proxmox cluster.

---

## 🔧 Step 1: Install Terraform

### On Linux/macOS:

```bash
sudo apt install unzip -y
wget https://releases.hashicorp.com/terraform/<VERSION>/terraform_<VERSION>_linux_amd64.zip
unzip terraform_*.zip
sudo mv terraform /usr/local/bin/
terraform -v
```

> 📸 **Photo Suggestion 2**: Screenshot of `terraform -v` output.

---

## 🔑 Step 2: Create Proxmox API Token

1. **Login to Proxmox Web UI**
2. Go to `Datacenter → Permissions → API Tokens`
3. Create a token for your user (e.g., `terraform@pve`)
4. Assign necessary privileges (e.g., `VM.Allocate`, `VM.Config.*`, `Datastore.AllocateSpace`, etc.)

> 📸 **Photo Suggestion 3**: Screenshot of API token creation dialog.

---

## 📁 Step 3: Terraform Project Structure

```bash
terraform-proxmox-cluster/
├── main.tf
├── variables.tf
├── outputs.tf
└── terraform.tfvars
```

---

## ✍️ Step 4: main.tf – Proxmox Provider and VM Resource

```hcl
terraform {
  required_providers {
    proxmox = {
      source  = "telmate/proxmox"
      version = "~> 2.9.14"
    }
  }
}

provider "proxmox" {
  pm_api_url = "https://192.168.1.10:8006/api2/json"
  pm_api_token_id = "terraform@pve!mytoken"
  pm_api_token_secret = var.pm_api_token_secret
  pm_tls_insecure = true
}

resource "proxmox_vm_qemu" "vm1" {
  name = "tf-vm01"
  target_node = "pve-node1"
  clone = "ubuntu-template"
  cores = 2
  memory = 2048
  net0 = "virtio,bridge=vmbr0"
  os_type = "cloud-init"
  sshkeys = file("~/.ssh/id_rsa.pub")
  ciuser = "ubuntu"
  ipconfig0 = "ip=192.168.1.101/24,gw=192.168.1.1"
  agent = 1
  disk {
    size = "10G"
    type = "scsi"
    storage = "local-lvm"
  }
}
```

> 📸 **Photo Suggestion 4**: VSCode screenshot of main.tf opened with syntax highlighting.

---

## 📦 Step 5: variables.tf

```hcl
variable "pm_api_token_secret" {
  type = string
  description = "Proxmox API token secret"
  sensitive = true
}
```

---

## 🧪 Step 6: Initialize and Apply

```bash
terraform init
terraform plan
terraform apply
```

- On `apply`, Terraform will connect to Proxmox API and spin up a VM from the template.

> 📸 **Photo Suggestion 5**: Terminal output of `terraform apply` showing successful creation.

---

## 🎯 Step 7: Verify in Proxmox Dashboard

- Log in to the **Proxmox web UI**
- You’ll see the VM `tf-vm01` running on `pve-node1`

> 📸 **Photo Suggestion 6**: Proxmox UI showing the newly created VM.

---

## 🔁 Reusability: Scaling to Multiple VMs

You can define a map or a loop to create multiple VMs dynamically:

```hcl
variable "vm_list" {
  type = list(string)
  default = ["tf-vm01", "tf-vm02", "tf-vm03"]
}

resource "proxmox_vm_qemu" "multi" {
  count = length(var.vm_list)
  name = var.vm_list[count.index]
  ...
}
```

> 📸 **Photo Suggestion 7**: Screenshot showing 3 VMs created via loop.

---

## 🔒 Best Practices

- Use **`.tfvars`** and **`.env`** files for secrets.
- Enable **TLS** and avoid disabling SSL verification in production.
- Version control your configs with Git.
- Create **cloud-init** templates to streamline provisioning.
- Use **modules** for reusable code across environments.

---

## 🧰 Advanced: Using Modules

Create a reusable module:

```hcl
module "ubuntu_vm" {
  source = "./modules/ubuntu"
  name = "tf-vm04"
  ip = "192.168.1.104"
}
```

This makes your infrastructure DRY and modular.

---

## 🧾 Conclusion

By integrating **Terraform** with **Proxmox VE**, you've added **repeatability, automation**, and **version control** to your home lab or test environment. This approach not only improves productivity but also prepares your environment for professional-grade scalability.

---

## 📸 Suggested Photos Summary

| Step | Image                                         |
| ---- | --------------------------------------------- |
| 1    | Diagram of Terraform interacting with Proxmox |
| 2    | Terraform version check                       |
| 3    | Proxmox API token creation                    |
| 4    | Screenshot of `main.tf`                       |
| 5    | `terraform apply` output                      |
| 6    | Proxmox UI with Terraform VM                  |
| 7    | Loop-created multiple VMs in dashboard        |

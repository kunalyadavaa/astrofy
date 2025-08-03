---
title: "Proxmox VE Cluster"
description: "**Proxmox VE** (Virtual Environment) is a powerful open-source virtualization platform based on Debian, allowing the management of KVM virtual machines and LXC containers. One of its greatest strengths is the ability to **create a cluster** of nodes, enabling **centralized management, high availability (HA)**, and **live migration**."
pubDate: "Feb 25 2025"
heroImage: "/proxmox.webp"
tags: ["proxmox", VE, cluster, Homelab]
---

# 🖥️ Building a Proxmox VE Cluster for Home Lab or Production Use

## 🔧 Introduction

In this project, we’ll walk through the **end-to-end setup** of a Proxmox VE Cluster, including installation, configuration, and optional features like Ceph, shared storage, and HA.

---

## 🛠️ Prerequisites

Before we begin, ensure you have:

| Component | Specification                                            |
| --------- | -------------------------------------------------------- |
| Nodes     | 2 or more physical machines (x86_64, VT-x/AMD-V enabled) |
| Network   | Static IPs on the same subnet, ideally with 1Gbps+       |
| Storage   | Local SSDs/HDDs, or shared NFS/Ceph                      |
| USB/DVD   | For Proxmox ISO installation                             |
| Software  | [Proxmox VE ISO](https://www.proxmox.com/en/downloads)   |

> 📸 **Photo Suggestion 1**: Your home lab setup or a simple network diagram with 3 Proxmox nodes and a router.

---

## 🧩 Step 1: Install Proxmox VE on All Nodes

### 🔹 Installation Steps

1. Flash the Proxmox ISO to a USB drive.
2. Boot each node from the USB.
3. Follow the on-screen instructions:

   - Set hostname (e.g., `pve-node1.local`)
   - Assign a **static IP**
   - Choose storage (ZFS is recommended for redundancy)

4. Reboot and access Proxmox at `https://<node-ip>:8006`

> 📸 **Photo Suggestion 2**: Screenshot of the Proxmox installer partition screen.

---

## 🔗 Step 2: Set Up SSH and Hostnames

1. Add all node hostnames to `/etc/hosts` on each machine:

```bash
192.168.1.10  pve-node1
192.168.1.11  pve-node2
192.168.1.12  pve-node3
```

2. Test SSH connectivity:

```bash
ssh root@pve-node2
```

> 📸 **Photo Suggestion 3**: Terminal showing SSH access between nodes.

---

## 🧠 Step 3: Create the Cluster on the First Node

On `pve-node1`:

```bash
pvecm create my-cluster
```

- This initializes the cluster and sets it as the master node.

Check the cluster status:

```bash
pvecm status
```

> 📸 **Photo Suggestion 4**: Cluster status output showing “Quorum OK”.

---

## ➕ Step 4: Join Other Nodes to the Cluster

On `pve-node2` and `pve-node3`, run:

```bash
pvecm add 192.168.1.10
```

- Enter the root password of `pve-node1`.

Confirm with:

```bash
pvecm nodes
```

> 📸 **Photo Suggestion 5**: GUI dashboard showing all nodes in the cluster.

---

## 🧰 Step 5: Configure Shared Storage (Optional but Recommended)

**Option 1: NFS**

On a NAS or another Linux server, export an NFS share, then:

```bash
Datacenter → Storage → Add → NFS
```

**Option 2: Ceph Cluster**

If you want high availability with redundancy:

1. Install Ceph via GUI (`Datacenter → Ceph`).
2. Create Monitors and OSDs.
3. Set up CephFS or RBD storage.

> 📸 **Photo Suggestion 6**: Screenshot of Proxmox GUI adding NFS or Ceph storage.

---

## 🔁 Step 6: Enable HA and Live Migration

1. Ensure **shared storage** is available to all nodes.
2. Migrate VM: Right-click → **Migrate** → Select node.
3. Configure HA via **Datacenter → HA**.

> 📸 **Photo Suggestion 7**: HA resource config page or live migration progress.

---

## 🎛️ Optional: Web UI Dashboard Setup

Install additional packages for better metrics:

```bash
apt install ifupdown2 glances htop
```

Use Proxmox’s **cluster-wide dashboard** to monitor load, RAM, uptime, etc.

> 📸 **Photo Suggestion 8**: Screenshot of the Proxmox Cluster Summary Dashboard.

---

## 🔐 Optional: Backup, Firewall & Updates

### 🔹 Backups

- Configure scheduled backups using:

  - **PBS (Proxmox Backup Server)**
  - Local or NFS-based storage

### 🔹 Firewall

- Configure at Datacenter or Node level.
- Use `iptables` or built-in GUI rules.

### 🔹 Updates

```bash
apt update && apt full-upgrade
```

> 📸 **Photo Suggestion 9**: Backup configuration screen or update CLI output.

---

## ✅ Conclusion

You now have a **functional Proxmox VE cluster** capable of managing your VMs and containers efficiently with centralized control, HA, and optional redundancy. This setup is ideal for learning, testing, or running small services in production.

---

## 📂 Future Enhancements

- Add a Proxmox Backup Server (PBS)
- Integrate Zabbix/Prometheus for monitoring
- Automate with Ansible or Terraform
- VLAN segmentation for virtual networks
- Enable 2FA and Let's Encrypt for secure access

---

## 📸 Appendix: Suggested Image Captures

| Step | Description                        |
| ---- | ---------------------------------- |
| 1    | Lab setup/network diagram          |
| 2    | Proxmox ISO installer screen       |
| 3    | SSH terminal between nodes         |
| 4    | pvecm cluster status               |
| 5    | Proxmox GUI showing 3-node cluster |
| 6    | Storage addition (NFS/Ceph)        |
| 7    | HA migration example               |
| 8    | Cluster summary dashboard          |
| 9    | Backup settings or update commands |

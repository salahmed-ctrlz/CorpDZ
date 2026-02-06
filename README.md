# 🏢 CORP.DZ — Enterprise Infrastructure Home Lab

![Project Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge\&logo=github)
![Environment](https://img.shields.io/badge/Type-Home_Lab-blue?style=for-the-badge)
![Role](https://img.shields.io/badge/Role-SysAdmin_%2F_Network_Engineer-orange?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-Oracle_VirtualBox-green?style=for-the-badge)

> **A hybrid enterprise network simulation that replicates SMB infrastructure.** This lab demonstrates on‑premise integration of Windows Active Directory and Linux services, identity management, automation, defensive auditing, and basic offensive testing.

---

## 📑 Table of Contents

* [Project Overview](#-project-overview)
* [Author & Links](#-author--links)
* [Laboratory Environment](#-laboratory-environment)
* [Network Topology](#-network-topology)
* [Implementation Walkthrough](#-implementation-walkthrough)

  * [Phase 1: The Foundation (Identity & DNS)](#phase-1-the-foundation-identity--dns)
  * [Phase 2: Hybrid Services (Linux & Web)](#phase-2-hybrid-services-linux--web)
  * [Phase 3: Operations & Security (GPO & Audit)](#phase-3-operations--security-gpo--audit)
* [Tools & Utilities](#-tools--utilities)
* [Key Configurations](#-key-configurations)
* [Usage & Validation](#-usage--validation)
* [Disclaimer](#-disclaimer)

---

## 🔍 Project Overview

**Corp.DZ** is a hands‑on home lab built to emulate a Small‑to‑Medium Business (SMB) on‑premises environment. The lab emphasizes practical, reproducible configuration of networking and identity services without cloud abstractions.

**Primary goals:**

* Build an **air‑gapped internal LAN** for safe testing.
* Integrate **Windows Server (AD/DNS/DHCP)** with **Ubuntu Linux web services**.
* Automate user provisioning and device configuration using Group Policy and PowerShell.
* Validate security posture via policy enforcement and an internal Kali audit.
* Produce reproducible documentation and artifacts for interviews and portfolio.

---

## 👨‍💻 Author & Links

**Salah Eddine Medkour** — Junior Network Engineer | Master’s in Networks & Telecommunications

* Portfolio: [https://salahmed-ctrlz.github.io/salaheddine-medkour-portfolio/](https://salahmed-ctrlz.github.io/salaheddine-medkour-portfolio/)
* LinkedIn: [https://www.linkedin.com/in/salah-eddine-medkour/](https://www.linkedin.com/in/salah-eddine-medkour/)
* GitHub: [https://github.com/salahmed-ctrlz/](https://github.com/salahmed-ctrlz/)

---

## 🛠️ Laboratory Environment

Host runs Oracle VirtualBox with multiple VMs connected to internal virtual networks to simulate an isolated corporate LAN.

### Host Hardware (reference)

* CPU: AMD Ryzen 5 5600 (6C/12T)
* RAM: 16 GB DDR4
* Storage: NVMe SSD
* Network: 1.5 Gbps FTTH (used only for ISO and package retrieval)

### VM Inventory

| Hostname      |                                       Role | OS                         | IP / Network          |
| ------------- | -----------------------------------------: | -------------------------- | --------------------- |
| CorpDZ-DC     | Domain Controller (AD/DNS/DHCP/FileServer) | Windows Server 2022        | Static `192.168.10.2` |
| CorpDZ-Client |                                Workstation | Windows 10 Enterprise LTSC | DHCP (.100-.200)      |
| CorpDZ-Web    |                        Intranet Web Server | Ubuntu Server 24.04        | Static `192.168.10.5` |
| Kali-Audit    |                          Security Audit VM | Kali Linux Rolling         | DHCP                  |

---

## 🌐 Network Topology

Single internal network used for Lab 1:

```
Corp.DZ Internal Network — 192.168.10.0/24
  - CorpDZ-DC (192.168.10.2) — AD, DNS, DHCP, File Share
  - CorpDZ-Web (192.168.10.5) — Nginx intranet
  - CorpDZ-Client — Domain member, receives DHCP
  - Kali-Audit — Recon & scans
```

(For Lab 2, the topology will be segmented with multiple subnets and routing.)

---

## 📝 Implementation Walkthrough

### Phase 1: The Foundation (Identity & DNS)

* Installed Windows Server 2022 and configured static networking (`192.168.10.2`).
* Installed AD DS, promoted to DC for forest `corp.dz`.
* Verified DNS auto‑installation and set DNS to loopback on the DC.
* Deployed DHCP scope `OfficeLAN` (range examples used in lab). Client PC successfully joined the domain and received an IP via DHCP.

**Notes / Gotchas:** ensure clients point DNS to the DC; otherwise name resolution and domain join fail.

---

### Phase 2: Hybrid Services (Linux & Web)

* Deployed Ubuntu Server 24.04 and configured static addressing with Netplan.
* Installed Nginx and deployed a simple intranet site `http://www.corp.dz`.
* Created a Windows DNS A record mapping `www.corp.dz` → `192.168.10.5`.
* Verified cross‑platform resolution from Windows clients.

---

### Phase 3: Operations & Security (GPO & Audit)

* Implemented OU structure and RBAC with delegated helpdesk permission for a Sales OU.
* Used Group Policy to map drives, enforce desktop wallpaper, redirect folders, and deploy MSI packages (7‑Zip) via Computer Configuration.
* Hardened Default Domain Policy (password complexity, account lockout threshold configured and validated).
* Performed internal reconnaissance with Kali (netdiscover, nmap) to confirm attack surface and open services for defensive assessment.

---

## 🧰 Tools & Utilities

* Custom: **99SAK — PowerShell Swiss Army Knife** (admin helpers and triage scripts) — [https://github.com/salahmed-ctrlz/99SAK-PowershellSwissArmyKnife](https://github.com/salahmed-ctrlz/99SAK-PowershellSwissArmyKnife)
* RSAT (Remote Server Administration Tools)
* PuTTY / OpenSSH for Linux administration
* Nmap / Netdiscover for network discovery and audit
* VirtualBox for VM orchestration

---

## 🔧 Key Configurations (Examples)

### Ubuntu Netplan (`/etc/netplan/50-cloud-init.yaml`)

```yaml
network:
  ethernets:
    enp0s3:
      addresses: [192.168.10.5/24]
      routes:
        - to: default
          via: 192.168.10.1
      nameservers:
        addresses: [192.168.10.2]
  version: 2
```

### PowerShell: Bulk User Import (snippet)

```powershell
$Users = Import-Csv "C:\HR_Data.csv"
ForEach ($User in $Users) {
    New-ADUser -Name $User.Name `
               -GivenName $User.FirstName `
               -Path "OU=Sales,DC=corp,DC=dz" `
               -AccountPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) `
               -Enabled $true
}
```

### Folder Redirection Strategy

* GPO Path: `User Configuration > Policies > Windows Settings > Folder Redirection`
* Target UNC (IP‑hardcoded): `\\192.168.10.2\UserFiles\%USERNAME%\Desktop`

  * Rationale: using IP reduces boot timing dependency on DNS services.

---

## ✅ Usage & Validation

* **Validation steps performed:**

  * DNS resolution from client to `www.corp.dz` (HTTP success)
  * Account lockout simulation & verification
  * ACL enforcement on sensitive share (Salaries folder)
  * Silent MSI deployment test via GPO — validated on machine boot
  * Reconnaissance scan from Kali to verify expected open services

* **Deliverables:**

  * Technical diary (ticket log)
  * PowerShell automation scripts
  * Screenshots and evidence (available in portfolio)

---

## 📌 Disclaimer

This lab is a home‑lab simulation for educational and portfolio use. Exercise caution when applying similar configurations to production environments. The author is not responsible for misuse.

---

© 2026 Salah Eddine Medkour

---

*Ready for Lab 2 (Network segmentation & routing).*

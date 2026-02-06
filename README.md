

```markdown
# 🏢 Corp.DZ Enterprise Infrastructure Lab

![Project Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge&logo=github)
![Environment](https://img.shields.io/badge/Type-Home_Lab-blue?style=for-the-badge)
![Role](https://img.shields.io/badge/Role-SysAdmin_%2F_Network_Engineer-orange?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-Oracle_VirtualBox-green?style=for-the-badge)

> **A hybrid enterprise network simulation designed to replicate Small-to-Medium Business (SMB) infrastructure. This project explores the integration of Windows Active Directory with Linux services, identity management, automation, and defensive/offensive security auditing.**

---

## 📑 Table of Contents
- [Project Overview](#-project-overview)
- [Author & Links](#-author--links)
- [Laboratory Environment](#-laboratory-environment)
- [Network Topology](#-network-topology)
- [Implementation Walkthrough](#-implementation-walkthrough)
  - [Phase 1: The Foundation (Identity & DNS)](#phase-1-the-foundation-identity--dns)
  - [Phase 2: Hybrid Services (Linux & Web)](#phase-2-hybrid-services-linux--web)
  - [Phase 3: Operations & Security (GPO & Audit)](#phase-3-operations--security-gpo--audit)
- [Tools & Utilities](#-tools--utilities)
- [Key Configurations](#-key-configurations)
- [Disclaimer](#-disclaimer)

---

## 🔍 Project Overview
**Corp.DZ** is a simulated corporate environment built from the ground up to demonstrate full-stack infrastructure management. Unlike cloud-based labs, this project focuses on **on-premise logic**, requiring manual configuration of TCP/IP stacks, DNS zones, and Active Directory forests.

**Key Objectives Achieved:**
* **Air-Gapped Architecture:** Built a completely isolated internal subnet (`192.168.10.0/24`) to simulate a secure LAN.
* **Hybrid OS Integration:** Successfully bridged **Windows Server 2022** and **Ubuntu Linux 24.04** environments using internal DNS A-Records.
* **Identity Management:** Deployed **Active Directory Domain Services (AD DS)** with a custom Forest, complex OU structures, and Role-Based Access Control (RBAC).
* **Disaster Recovery:** Implemented **Folder Redirection** to decouple user data from client hardware.

---

## 👨‍💻 Author & Links
**Salah Eddine Medkour** *Junior Network Engineer | Master’s in Networks & Telecommunications*

* 🌐 **Portfolio:** [salahmed-ctrlz.github.io](https://salahmed-ctrlz.github.io/salaheddine-medkour-portfolio/)
* 💼 **LinkedIn:** [Salah Eddine Medkour](https://www.linkedin.com/in/salah-eddine-medkour/)
* 🐙 **GitHub:** [salahmed-ctrlz](https://github.com/salahmed-ctrlz/)

---

## 🛠️ Laboratory Environment
The infrastructure operates on a high-performance local workstation using **Oracle VirtualBox**. VMs are bridged via a dedicated internal virtual adapter to ensure network isolation.

### **Host Hardware Configuration**
* **CPU:** AMD Ryzen 5 5600 (6 Cores / 12 Threads)
* **GPU:** NVIDIA GTX 1650
* **RAM:** 16 GB DDR4 (Allocated dynamically to VMs)
* **Storage:** NVMe SSD (Critical for concurrent VM I/O performance)
* **Network:** 1.5 Gbps FTTH (For ISO retrieval and updates)
* **Display:** Dual Monitor Setup (Admin Center / Documentation)

### **Virtual Machine Inventory (BOM)**
| Hostname | Role | OS Version / ISO Used | Network Config |
| :--- | :--- | :--- | :--- |
| **CorpDZ-DC** | **Domain Controller** | Windows Server 2022 (Eval) | Static IP: `192.168.10.2` |
| **CorpDZ-Client** | **Workstation** | Windows 10 Enterprise LTSC 2021 | DHCP (Scope: `.100-.200`) |
| **CorpDZ-Web** | **Intranet Server** | Ubuntu Server 24.04 LTS | Static IP: `192.168.10.5` |
| **Kali-Audit** | **Security Audit** | Kali Linux Rolling 2024 | DHCP |

---

## 🌐 Network Topology
The lab simulates a "Branch Office" single-subnet architecture.

```mermaid
graph TD
    subgraph "Corp.DZ Internal Network (192.168.10.0/24)"
        DC[CorpDZ-DC (Win Srv 2022)] -- DNS/DHCP/AD --> SW(Virtual Switch)
        WEB[Ubuntu-Web (Nginx)] -- HTTP Port 80 --> SW
        CLIENT[PC-CEO (Win 10)] -- Domain Member --> SW
        KALI[Kali-Audit] -- Netdiscover/Nmap --> SW
    end

```

---

## 📝 Implementation Walkthrough

### Phase 1: The Foundation (Identity & DNS)

*Objective: Establish the "Data Center" core and onboard the first client.*

* **Server Provisioning:** Installed Windows Server 2022 Standard.
* **Domain Promotion:** Promoted server to Domain Controller; established Root Forest `corp.dz`.
* **IP Management:**
* Configured Static IP (`192.168.10.2`) on the DC.
* Authorized **DHCP Scope** `OfficeLAN` to serve dynamic IPs to workstations.


* **Client Onboarding:** Joined a Windows 10 LTSC workstation to the domain.
* **Troubleshooting:** Resolved DNS resolution failures by ensuring Client DNS pointed exclusively to `192.168.10.2` (Split-Brain DNS concept).

### Phase 2: Hybrid Services (Linux & Web)

*Objective: Integrate Open Source infrastructure into the Windows ecosystem.*

* **Linux Deployment:** Deployed **Ubuntu Server 24.04**. Configured networking manually via **Netplan** (YAML) to ensure static addressing.
* **Web Server:** Installed **Nginx** (LAMP stack component) and deployed a custom "Authorized Personnel Only" HTML dashboard.
* **Cross-Platform Resolution:** Created a **DNS A-Record** (`www`) on the Windows Server pointing to the Linux IP (`192.168.10.5`).
* *Result:* Windows clients can resolve `http://www.corp.dz` internally.



### Phase 3: Operations & Security (GPO & Audit)

*Objective: Automate management tasks and harden the network security posture.*

* **Least Privilege Access:**
* Created a tiered administrative account (`Adam.Helpdesk`).
* Used **Delegation of Control** to grant "Password Reset" rights *only* for the Sales OU, preventing privilege escalation to Domain Admin.


* **Group Policy Architecture:**
* **The "Wallpaper War":** Demonstrated **GPO Precedence (LSDOU)** by creating conflicting policies (Domain Level vs. OU Level).
* **Software Deployment:** Automated the installation of **7-Zip** via MSI package assignment.


* **Data Assurance (Folder Redirection):**
* Configured GPO to redirect user `Desktop` & `Documents` to a central share (`\\CorpDZ-DC\UserFiles`).
* *Technical Fix:* Hardcoded the IP in the UNC path to bypass potential NetBIOS naming issues during boot.


* **Offensive Audit:**
* Deployed **Kali Linux** to perform internal reconnaissance.
* Used `netdiscover` and `nmap` to map the attack surface and verify exposed ports (53, 88, 389, 445).



---

## 🧰 Tools & Utilities

Primary tools leveraged for management, automation, and troubleshooting:

* **[99SAK - Powershell Swiss Army Knife](https://github.com/salahmed-ctrlz/99SAK-PowershellSwissArmyKnife)**
* *A custom single-file, portable admin toolkit I developed. Used extensively for fast on-host triage, log analysis, and routine system operations during the lab.*


* **RSAT (Remote Server Administration Tools):** Managed AD users from the Client workstation to simulate real-world admin workflows.
* **PuTTY:** SSH management for the Ubuntu Web Server.
* **Nmap:** Network mapping and service enumeration.

---

## 💻 Key Configurations

### 1. Ubuntu Netplan Config (`/etc/netplan/50-cloud-init.yaml`)

*Critical configuration for pointing Linux DNS to the Windows Controller.*

```yaml
network:
  ethernets:
    enp0s3:
      addresses: [192.168.10.5/24]
      routes:
        - to: default
          via: 192.168.10.1
      nameservers:
        addresses: [192.168.10.2] # Points to Windows AD DNS
  version: 2

```

### 2. PowerShell Bulk User Import (Concept Snippet)

*Used to automate HR onboarding.*

```powershell
# Import CSV and loop through each row to create AD Users
$Users = Import-Csv "C:\HR_Data.csv"
ForEach ($User in $Users) {
    New-ADUser -Name $User.Name `
               -GivenName $User.FirstName `
               -Path "OU=Sales,DC=corp,DC=dz" `
               -AccountPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) `
               -Enabled $true
}

```

### 3. Folder Redirection Strategy

* **GPO Setting:** `User Configuration > Policies > Windows Settings > Folder Redirection`
* **Target Path:** `\\192.168.10.2\UserFiles\%USERNAME%\Desktop`
* **Rationale:** Using the IP address in the UNC path ensures the policy applies even if DNS services are slow to start during the boot sequence.

---

## 📌 Disclaimer

This project is a **Home Lab simulation** intended for educational purposes and skill development. It utilizes enterprise-grade software and configurations (AD, GPO, Linux Servers) to replicate a production environment.

---

*© 2026 Salah Eddine Medkour. Documented as part of the "Zero-to-Hero" Infrastructure Sprint.*

```

```


---

Turn to proper MD (expanded)

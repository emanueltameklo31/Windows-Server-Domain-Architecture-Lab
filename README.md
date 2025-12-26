# Windows-Server-Domain-Architecture

## 📌 Overview
This project was designed and deployed to simulate an enterprise Active Directory environment with centralized identity and access management, secure network services (DNS, DHCP, NAT), and domain-joined endpoints, serving as a foundation for cybersecurity monitoring, privilege management, and identity-based threat scenarios. This kind of enviroment is what you’d expect to see in corporate IT, government networks, and DoD-aligned organizations.

---

## 🏗️ What Will Be Created

### 🖥️ Domain Controller (Windows Server 2019)
- Configured with **two network interfaces (NICs)**:
  - **NIC 1 – NAT:** Provides controlled internet access
  - **NIC 2 – Internal Network:** Isolated private lab network
- Hosts core enterprise services:
  - Active Directory Domain Services (AD DS)
  - DNS
  - DHCP
  - Routing and Remote Access (RAS/NAT)

### 💻 Client Workstation (Windows 10)
- Domain-joined Windows 10 virtual machine
- Connected exclusively to the **internal network**
- Accesses the internet securely **through the Domain Controller**

### Network Architecture Overview

 Internet 
     |
    NAT 
     |
 Domain Controller 
     |
 Internal Network 
     |
 Windows 10 Client 

---

## 1) Downloads (one-time)

⭐ **Goal**: Prepare the virtualization and operating system resources required to simulate an enterprise Windows domain environment.

1. Install **Oracle VirtualBox** 
2. Download ISOs:
   * **Windows Server 2019** You can use this link: https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019
   * **Windows 10 ISO** You can use this link: https://www.microsoft.com/en-us/software-download/windows10
- Download the 64-bit .iso file for both images.

---

## 2) Create the Domain Controller VM (Server 2019)

⭐ **Goal**: Create a dedicated virtual server to act as the centralized identity, authentication, and network services host.

1. Open **VirtualBox → New**
2. Name: `DC` (or `Domain Controller`)
3. Type: **Microsoft Windows**
4. Version: **Windows 2019 (64-bit)**
5. Memory: **2 GB+** (more if your PC allows)
6. Disk: **40–60 GB** (dynamic is fine)
7. Mount the **Server 2019 ISO** as the boot media

### Add the 2 network adapters (this part matters a lot)

1. Select the DC VM → **Settings → Network**
2. **Adapter 1**: Enable → **Attached to: NAT**
3. **Adapter 2**: Enable → **Attached to: Internal Network**
   * Name it something like: `intnet` (any name is fine, just make sure you can remember it)

---

## 3) Install Windows Server 2019

⭐ **Goal**: Deploy a secure server operating system to support enterprise identity and infrastructure services.

1. Start the DC VM
2. Install **Windows Server 2019 Standard (Desktop Experience)** 
3. Create the local Administrator password and sign in (I would suggest "Password1") 

---

## 4) Name the server + set the Internal NIC static IP

⭐ **Goal**: Ensure consistent identification and reliable network communication for domain and DNS services.

### A) Rename the server
- Server Manager → Local Server → Computer name → rename to something like `DC` → reboot

### B) Identify the two NICs and rename them (recommended)
- Control Panel → Network & Internet → Network Connections
- Rename:
  * NAT-facing NIC → `INTERNET`
  * Internal NIC → `INTERNAL`

### C) Set static IPv4 on INTERNAL NIC

On the **INTERNAL** adapter:

- IP: **172.16.0.1**
- Subnet mask: **255.255.255.0**
- Default gateway: *(leave blank)* (typical for this lab)
- DNS: **127.0.0.1** (or **172.16.0.1** after DNS is up)

---

## 5) Install Active Directory Domain Services (and DNS) + promote to Domain Controller

⭐ **Goal**: Implement centralized identity, authentication, and authorization for the enterprise environment.

1. Server Manager → **Add Roles and Features**
2. Role: **Active Directory Domain Services (AD DS)**
   * (DNS typically comes along / is added during promotion)
3. After install: click the flag notification → **Promote this server to a domain controller**
4. Choose **Add a new forest**
5. Domain name (example): `mydomain.com`
6. Set the Directory Services Restore Mode password → Install → Reboot

---

## 6) Create an admin Organizational Unit + a “Domain Admin” user (good practice)

⭐ **Goal**: Enforce role separation and least-privilege principles for administrative access.

1. Open **Active Directory Users and Computers (ADUC)**
2. Create an Organizational Unit (OU) like:

   * `_ADMINS`
3. Create a user (example: `a-initial of first name + lastname` i.e a-jdoe for john doe)
4. Add user to:

   - **Domain Admins** (and optionally Enterprise Admins for lab use)

---

## 7) Enable internet for the internal network (RAS/NAT)

⭐ **Goal**: Client on Internal Network can browse the web through the DC.

1. Server Manager → Add Roles and Features
2. Install **Remote Access**
3. Under Role Services: select **Routing**
4. After install:
   - Tools → **Routing and Remote Access**
   - Right-click DC → **Configure and Enable Routing and Remote Access**
   - Choose **NAT** (or “custom configuration” → NAT)
   - Select the **INTERNET** interface as the public interface

---

## 8) Set up DHCP on the Domain Controller

⭐ **Goal**: Automate IP address assignment and support scalable endpoint onboarding.

1. Server Manager → Add Roles and Features → **DHCP Server**
2. After install: complete DHCP post-install config
3. In DHCP console:
   - Create a new IPv4 Scope:
     * Scope range: **172.16.0.100 – 172.16.0.200**
     * Mask: **255.255.255.0**
     * Router (gateway): **172.16.0.1**
     * DNS: **172.16.0.1** 

---

## 9) Bulk-create users with PowerShell (“1k users”)

⭐ **Goal**: Simulate enterprise-scale identity provisioning and demonstrate automation skills relevant to IAM and SOC operations.

Paste this link into Internet Explorer's URL:  https://github.com/joshmadakor1/AD_PS/archive/master.zip
1. Save it on to your desktop in your VM
2. Open **PowerShell as Administrator** on the DC
3. Run the script (edit variables first like password + count)
  - To allow script execution, in the command line type: Set-ExecutionPolicy Unrestricted
  - Navigate to script directory by typing: c:\users\your-admin-name\desktop\AD_PS-master then press the green play button above and your script will run
   
---

## 10) Create the Windows 10 Client VM and join it to the domain

⭐ **Goal**: Simulate an employee workstation in an enterprise domain.
 
### A) Create the VM
1. VirtualBox → **New**
2. Name: `CLIENT1`
3. Version: Windows 10 (64-bit)
4. RAM: **2–4 GB**
5. Disk: **40+ GB**
6. Install Windows 10 from ISO 

### B) Put CLIENT1 on the internal network
* VM Settings → Network
* Adapter 1: **Internal Network** (same name you used earlier, e.g., `intnet`) 

### C) Verify it gets DHCP + internet
On CLIENT1:
1. Open CMD → `ipconfig`
2. You should see:
   * IP like **172.16.0.x**
   * DNS **172.16.0.1**
3. Try browsing or pinging websites to confirm internet works (via DC NAT)

### D) Join the domain

1. System → Rename this PC (advanced) → **Change**
2. Select **Domain** → enter: `mydomain.com`
3. Provide domain admin creds → reboot
4. Try logging in as a domain user (username should be first inital of firstname + lastname (i.e jdoe for johndoe) and the password should be 'Password1'


## 🛠️ Skills Used
- Active Directory Domain Services (AD DS) – Domain creation, OU structure, user and group management
- Identity & Access Management (IAM) – Centralized authentication, role separation, and account provisioning
- Windows Server 2019 Administration – Server configuration, role installation, and infrastructure management
- Networking Fundamentals – DNS, DHCP, NAT, internal network segmentation
- Virtualization – VirtualBox VM design and multi-NIC configuration
- PowerShell – Bulk user creation and directory automation
- Cybersecurity Foundations – Identity-centric security, privilege management, and enterprise attack-surface awareness

## 🙌 Acknowledgment

This project’s architecture and workflow were inspired by **Josh Madakor's** Active Directory home lab series, which provides an excellent foundation for understanding enterprise identity, access management, and Windows infrastructure security.

🔗 **Reference:** https://www.youtube.com/@JoshMadakor

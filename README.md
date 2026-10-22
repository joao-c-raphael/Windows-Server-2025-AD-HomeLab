# Enterprise Active Directory & File Management Lab

![Windows Server 2025](https://img.shields.io/badge/Windows%20Server-2025-0078D6?style=for-the-badge&logo=windows-server&logoColor=white)
![Windows 11](https://img.shields.io/badge/Windows%2011-Enterprise-0078D7?style=for-the-badge&logo=windows11&logoColor=white)
![VMware Workstation](https://img.shields.io/badge/VMware-Workstation%20Pro-607D8B?style=for-the-badge&logo=vmware&logoColor=white)
![Active Directory](https://img.shields.io/badge/Active%20Directory-AD%20DS-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)

## 📌 Project Overview
This project demonstrates the design, deployment, and configuration of an enterprise Active Directory Domain Services (AD DS) infrastructure hosted on VMware Workstation Pro. 

The primary objective is to simulate a real-world corporate IT environment featuring:
- A centralized Windows Server 2025 Domain Controller (`FileServer01`).
- Domain network integration for Windows 11 Enterprise clients.
- Role-Based Access Control (RBAC) through tailored Organizational Units (OUs), Security Groups, and NTFS/Share permissions.
- Hardened domain-wide security using enforced Group Policy Objects (GPOs).

---

## 📐 Network & Infrastructure Architecture

| Component | Hostname | Operating System | IP Configuration | Role / Function |
| :--- | :--- | :--- | :--- | :--- |
| **Domain Controller** | `FileServer01` | Windows Server 2025 | Static | Primary DC, DNS, File Server |
| **Client Workstation** | `Client01` | Windows 11 Enterprise | DHCP (DNS pointing to DC) | Domain-Joined Workstation |
| **Domain Name** | `SeafoodKing.local` | — | — | Active Directory Forest Root |

---

## 🛠️ Step-by-Step Implementation

### Phase 1: Virtual Infrastructure & Base OS Installation
1. Provisioned a Virtual Machine in **VMware Workstation Pro** using the Windows Server 2025 ISO.
2. Configured base system attributes:
   - Configured local Administrator account.
   - Updated system hostname to **`FileServer01`**.
3. Enabled **Remote Management** via Server Manager to support centralized administrative access.
4. Installed all outstanding **Windows Updates** prior to domain promotion to ensure baseline patch compliance and mitigate zero-day risks.

---

### Phase 2: Active Directory Domain Services Deployment
1. Added the **Active Directory Domain Services (AD DS)** and **DNS Server** roles via Server Manager.
   <img width="1012" height="797" alt="image" src="https://github.com/user-attachments/assets/e7d13eb9-1bc9-4e33-9a2e-d120089c0381" />
2. Promoted `FileServer01` to a Domain Controller for a new forest:
   - **Root Domain Name:** `SeafoodKing.local`
3. Configured static IPv4 networking parameters on the Domain Controller network interface.

---

### Phase 3: Active Directory Hierarchy & Client Onboarding

#### Organizational Unit (OU) & User Architecture
- Launched **Active Directory Users and Computers** (`dsa.msc`).
- Created a structured OU hierarchy representing corporate regional branches and operational departments (e.g., HR, Finance, IT).
- Provisioned domain user accounts with proper group assignments, delegating elevated privileges to designated Domain Admin accounts.
<img width="1013" height="802" alt="image" src="https://github.com/user-attachments/assets/58bd915b-82cd-4f66-8733-43508fa22d98" />

#### Windows 11 Workstation Integration
1. Deployed a Windows 11 Enterprise virtual machine.
2. Updated Client IPv4 DNS settings to point directly to `FileServer01`'s static IP.
   <img width="1017" height="797" alt="image" src="https://github.com/user-attachments/assets/aa47d27a-3271-4ef9-a160-5a3b56f9477c" />
3. Joined the workstation to the `SeafoodKing.local` domain.
   <img width="1015" height="802" alt="image" src="https://github.com/user-attachments/assets/ceb7cb67-28f1-4c3c-9857-9596c88b7b84" />
4. Relocated the newly added computer object into its respective departmental OU.

---

### Phase 4: Network File Share & Role-Based Access Control (RBAC)

#### Storage & Security Group Structure
1. Created a centralized data directory on the host storage: `C:\CompanyData`.
2. Structured departmental sub-directories (e.g., `C:\CompanyData\HR`, `C:\CompanyData\Finance`).
3. Created dedicated Active Directory **Security Groups** corresponding to each department and assigned respective users.
   <img width="1017" height="803" alt="image" src="https://github.com/user-attachments/assets/c95c95f3-954d-42a2-80e5-d601cc1df46f" />
   
#### Permission Model Configuration
- **NTFS Permissions:**
  - Explicitly configured NTFS permissions per department directory (e.g., granting `Modify` permissions strictly to the `HR` group while explicitly revoking non-essential access).
    <img width="1022" height="802" alt="image" src="https://github.com/user-attachments/assets/5f6c2d60-d9c3-4f92-b546-5e69f17049d0" />
  - Disabled permission inheritance where necessary to enforce strict confidentiality.
- **Share Permissions:**
  - Configured **Advanced Sharing** on `C:\CompanyData` with appropriate network access thresholds.
    <img width="1025" height="807" alt="image" src="https://github.com/user-attachments/assets/6a8c9b25-83b5-44e0-a213-5fcd2e84cb80" />
    
---

### Phase 5: Hardening Security via Group Policy Management

Using the **Group Policy Management Console** (`gpmc.msc`), enforced company-wide security standards across targeted OUs:

* 🔐 **Domain Password Policy**
  - Configured account lockouts, minimum password length, complexity standards, and password expiration rules.
    <img width="778" height="560" alt="image" src="https://github.com/user-attachments/assets/a247e125-19f6-4550-a87c-33367b4e6b38" />
* 🖼️ **Standardized Desktop Wallpaper Policy**
  - Enforced a uniform corporate background image across all domain-joined desktop instances (`User Configuration`).
* 🚫 **Control Panel Restriction**
  - Prevented non-administrative users from modifying system configurations by enabling *Prohibit access to Control Panel and PC settings*.
* 💾 **Removable Storage Device Access Control**
  - Prevented potential data exfiltration and USB malware entry by enforcing *Deny all access to removable storage classes*.
    <img width="1005" height="665" alt="image" src="https://github.com/user-attachments/assets/99e57c83-4ccb-453c-8120-68912e06a42c" />

---

## 🔍 Validation & Testing
- ✅ **Domain Connectivity:** Successfully logged into domain accounts on the Windows 11 client using `SEAFOODKING\<username>`.
- ✅ **DNS Resolution:** Verified name resolution using `nslookup SeafoodKing.local` and `ping FileServer01`.
- ✅ **Access Control Verification:** Confirmed users can only access SMB network shares corresponding to their assigned security groups.
- ✅ **GPO Enforcement:** Verified GPO application on client endpoints using `gpupdate /force` and `gpresult /r`.

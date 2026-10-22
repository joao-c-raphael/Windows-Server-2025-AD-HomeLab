# Windows Server 2025 Active Directory Home Lab

A hands-on IT support and systems administration project using Windows Server 2025, Active Directory Domain Services (AD DS), Group Policy, and a Windows 11 domain-joined client.

## Project Overview

This home lab simulates a small company's IT infrastructure. It demonstrates how to configure a Windows domain, manage users and computers, implement department-based file permissions, and apply centralized security policies.

**Objectives:**
- Install and configure Windows Server 2025 in a virtual machine.
- Deploy Active Directory Domain Services and create a domain.
- Organize users, computers, and departments using Organizational Units (OUs).
- Join a Windows 11 workstation to the domain.
- Configure shared folders and department-based access permissions.
- Implement Group Policy Objects (GPOs) for basic security and desktop management.
- Practice troubleshooting common Windows domain and client configuration issues.

## Lab Environment

| Component | Configuration |
|---|---|
| Host OS | Windows 11 |
| Virtualization | VMware Workstation Pro |
| Server OS | Windows Server 2025 |
| Client OS | Windows 11 |
| Directory Service | Active Directory Domain Services |
| Domain Name | `SeafoodKing.local` |
| Domain Controller | `FileServer01` |
| File Storage | `C:\CompanyData` |
| Management Tools | Server Manager, Active Directory Users and Computers, Group Policy Management Console |

> **Note:** This lab uses a single server for both domain controller and file server roles to keep the initial setup simple. A separate member server can be added in a future iteration.

## Network Architecture

The lab uses an isolated virtual network so the domain controller and client can communicate without exposing lab services to the public internet.

Example addressing plan:

| Device | Hostname | Example IP Address |
|---|---|---|
| Domain Controller | `FileServer01` | `192.168.10.10` |
| Windows 11 Client | `Client01` | `192.168.10.20` |
| DNS Server | `FileServer01` | `192.168.10.10` |
| Domain | — | `SeafoodKing.local` |

These addresses are examples. Select an unused subnet that matches your VMware virtual network configuration. Configure the domain controller with a static IP and configure the client to use the domain controller as its DNS server.

## Project Implementation

### 1. Install Windows Server 2025

1. Download the Windows Server 2025 evaluation ISO from Microsoft's official evaluation center.
2. Install VMware Workstation Pro.
3. Create a virtual machine and install Windows Server 2025.
4. Set a unique administrator password.
5. Rename the server to `FileServer01`.
6. Install Windows updates and restart as necessary.
7. Configure the server's network adapter with a static IP address.
8. Verify the hostname and network configuration.

**Validation:**
- Confirm the server has the correct hostname.
- Run `ipconfig /all` to verify its network configuration.
- Confirm the server can reach the intended network.

### 2. Configure Active Directory Domain Services

1. Open Server Manager.
2. Select **Add Roles and Features**.
<img width="1012" height="797" alt="image" src="https://github.com/user-attachments/assets/85bd6ffe-096b-448e-9516-613719e9f54c" />
<img width="1012" height="797" alt="image" src="https://github.com/user-attachments/assets/242e97f3-d711-4658-a176-b9dd2f357533" />
3. Install **Active Directory Domain Services (AD DS)**.
4. Complete the installation and select **Promote this server to a domain controller**.
5. Choose **Add a new forest**.
6. Set the root domain name to `SeafoodKing.local`.
7. Complete the domain controller configuration and restart the server.

**Validation:**
- Open Active Directory Users and Computers.
- Confirm the `SeafoodKing.local` domain appears.
- Verify that the domain controller's DNS service is functioning.

### 3. Create Organizational Units, Users, and Groups

Use Active Directory Users and Computers to create a basic organizational structure.

Example OU layout:

```text
SeafoodKing.local
├── Company
│   ├── Users
│   │   ├── HR
│   │   ├── IT
│   │   ├── Finance
│   │   └── Operations
│   ├── Computers
│   └── Groups
```

Implementation steps:

1. Create the required Organizational Units.
2. Create test user accounts for each department.
3. Create security groups for department-based access.
4. Add each user to the appropriate security group.
5. Keep standard user accounts separate from privileged administrative accounts.
6. Avoid granting Domain Admin privileges to ordinary employee accounts.

Example security groups:

| Security Group | Purpose |
|---|---|
| `GG_HR` | Access to HR resources |
| `GG_IT` | Access to IT resources |
| `GG_Finance` | Access to Finance resources |
| `GG_Operations` | Access to Operations resources |

**Validation:**
- Confirm that users and groups exist in the intended OUs.
- Verify group memberships before configuring file access.

### 4. Join a Windows 11 Client to the Domain

1. Create a second virtual machine.
2. Install Windows 11 using an appropriate installation image and license.
3. Connect the client to the same virtual network as the domain controller.
4. Configure the client's DNS server to use the domain controller's IP address.
5. Verify connectivity to the domain controller.
6. Open **Settings → System → About → Domain or workgroup** or the corresponding System Properties interface.
7. Join the computer to `SeafoodKing.local` using authorized domain credentials.
8. Restart the client.
9. Move the computer account into the appropriate Computers OU.

Useful commands:

```powershell
ipconfig /all
nslookup SeafoodKing.local
ping 192.168.10.10
```

Replace the example IP address with the actual address of your domain controller.

**Validation:**
- Confirm the client can resolve the domain through DNS.
- Sign in using a domain test account.
- Verify the computer account appears in Active Directory Users and Computers.

### 5. Configure Department File Shares and Permissions

Create the following directory structure on the server:

```text
C:\CompanyData
├── HR
├── IT
├── Finance
└── Operations
```

#### NTFS Permissions

1. Open the properties of each department folder.
2. Select the Security tab.
3. Assign the appropriate department security group the required permissions.
4. Grant only the access necessary for each department.
5. Review inherited permissions and remove unnecessary access carefully.
6. Preserve appropriate administrative and system access.

Example permissions:

| Folder | Security Group | Permission |
|---|---|---|
| `HR` | `GG_HR` | Modify |
| `IT` | `GG_IT` | Modify |
| `Finance` | `GG_Finance` | Modify |
| `Operations` | `GG_Operations` | Modify |

These are example assignments. Configure permissions for each folder and validate the effective access of test users.

#### Share Permissions

1. Open the properties of `C:\CompanyData`.
2. Select **Sharing → Advanced Sharing**.
3. Enable sharing and assign a share name, such as `CompanyData`.
4. Configure share permissions to support the intended access model.
5. Apply NTFS permissions to control access to individual department folders.

The shared path will follow this format:

```text
\\FileServer01\CompanyData
```

**Validation:**
- Test access with a standard HR account.
- Verify that the HR user can modify files in the HR folder.
- Verify that the HR user cannot access restricted Finance or IT folders.
- Repeat the tests for other departments.
- Check both share and NTFS permissions when troubleshooting access issues.

### 6. Implement Group Policy Objects

Use Group Policy Management to configure centralized settings for domain users and computers.

#### A. Domain Password Policy

1. Open Group Policy Management Console.
2. Edit the Default Domain Policy or another appropriately linked domain-level GPO.
3. Navigate to:

```text
Computer Configuration
└── Policies
    └── Windows Settings
        └── Security Settings
            └── Account Policies
                └── Password Policy
```

4. Configure password requirements based on your lab's security objectives.
5. Review account lockout settings as an additional security measure.

For standard domain accounts, domain account password policy is normally configured through a GPO linked at the domain level. Fine-Grained Password Policies are an option when different account groups require different password settings.

#### B. Desktop Wallpaper Policy

1. Create a shared location for the wallpaper image that domain users can access.
2. Create a GPO named `Desktop Wallpaper`.
3. Navigate to:

```text
User Configuration
└── Policies
    └── Administrative Templates
        └── Desktop
            └── Desktop
```

4. Configure the desktop wallpaper policy and specify a valid network path to the image.
5. Link the GPO to the appropriate user OU.

#### C. Restrict Control Panel Access

1. Create a GPO named `Restrict Control Panel`.
2. Navigate to:

```text
User Configuration
└── Policies
    └── Administrative Templates
        └── Control Panel
```

3. Enable **Prohibit access to Control Panel and PC settings**.
4. Link the GPO to a test user OU.

#### D. Restrict Removable Storage

1. Create a GPO named `Disable USB Storage`.
2. Navigate to:

```text
Computer Configuration
└── Policies
    └── Administrative Templates
        └── System
            └── Removable Storage Access
```

3. Configure the relevant removable-storage access policy.
4. Link the GPO to the appropriate computer OU.
5. Test with a non-administrative account and a lab USB storage device where available.

**Validation:**

Run the following commands on the client:

```powershell
gpupdate /force
gpresult /r
gpresult /h "$env:USERPROFILE\Desktop\gpresult.html"
```

Review the resulting Group Policy report to confirm which policies were applied.

## Troubleshooting and Verification

The following are useful scenarios to test and document during the project.

| Scenario | Diagnostic Approach |
|---|---|
| Client cannot join the domain | Check DNS settings, network connectivity, domain name resolution, and system time |
| Domain user cannot sign in | Verify account status, credentials, network connectivity, and domain controller availability |
| User cannot access a shared folder | Check share permissions, NTFS permissions, group membership, and effective access |
| Group Policy is not applying | Run `gpupdate /force` and review `gpresult` output |
| Domain cannot be resolved | Inspect DNS configuration and test with `nslookup` |
| Computer is missing from the intended OU | Locate the computer account in Active Directory and move it to the correct OU |

For each issue, record the symptoms, diagnostic commands, root cause, resolution, and validation results. Only document outcomes you have actually reproduced and verified.

## Security Considerations

- Use unique, strong passwords for administrative accounts.
- Do not reuse lab passwords on personal or production accounts.
- Keep the virtual lab isolated from production networks.
- Avoid exposing domain controller services directly to the internet.
- Use standard accounts for routine testing and reserve privileged accounts for administrative tasks.
- Apply the principle of least privilege to file and share permissions.
- Take VMware snapshots before major configuration changes.
- Use snapshots as lab recovery points, not as a substitute for proper backups.
- Use appropriately licensed Windows installation media and evaluation images.

## Skills Demonstrated

- Windows Server installation and configuration
- Active Directory Domain Services
- Domain controller promotion
- DNS configuration and domain name resolution
- User, group, and Organizational Unit administration
- Windows domain joining
- NTFS and shared-folder permissions
- Group Policy management
- Basic PowerShell diagnostics
- Windows client troubleshooting
- Access control and least-privilege principles
- Technical documentation

## Future Improvements

- Deploy a separate Windows Server member server for file services.
- Create a dedicated administrative account and practice delegated administration.
- Add a second client for multi-user testing.
- Configure a dedicated backup and recovery workflow.
- Document common help desk tickets and resolutions.
- Add screenshots of successful configuration and verification steps.
- Create a network diagram and an OU/group-permission diagram.
- Add a troubleshooting log with reproducible test cases.

## Project Status

**Status:** In progress

This project is a learning lab intended to demonstrate practical Windows administration and IT support skills. Configuration results and troubleshooting findings will be documented as the lab develops.

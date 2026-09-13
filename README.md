# Active Directory Home Lab

A fully functional Windows domain environment built from scratch using VirtualBox and Windows Server 2022. This lab demonstrates the core Active Directory skills required in help desk, IT support, and systems administrator roles.

---

## Lab Summary

| What | Detail |
|------|--------|
| Domain | `lab.local` |
| Domain Controller | Windows Server 2022 VM (DC02) |
| Organizational Units | IT, HR, Finance |
| User Accounts | 5 domain users |
| Security Groups | 3 department groups |
| Group Policy Object | IT-Security-Policy (password + lockout + USB block) |

---

## Environment

| Component | Detail |
|-----------|--------|
| Hypervisor | VirtualBox 7.2.14 |
| Server OS | Windows Server 2022 Standard Evaluation |
| ISO | `SERVER_EVAL_x64FRE_en-us.iso` |
| VM Name | DC02 |
| Host Machine | Lenovo IdeaPad, AMD Ryzen AI 5 340 |
| VM RAM | 4096 MB (4 GB) |
| VM Disk | 50 GB dynamic VDI |
| Domain Name | `lab.local` |

---

## Why I Built This

Active Directory is present in nearly every enterprise Windows environment. Understanding how to deploy a domain, manage users and groups, and enforce security policy through GPO is foundational for help desk, IT support, and sysadmin roles.

This lab was built entirely from scratch to practice:

- Deploying and configuring Windows Server from an evaluation ISO
- Installing and configuring the AD DS role
- Promoting a standalone server to a Domain Controller
- Structuring an organization in ADUC using Organizational Units
- Managing domain user accounts and security groups
- Creating and linking a Group Policy Object at the domain level
- Troubleshooting real errors encountered during the build

---

## Build Walkthrough

### Step 1 — Create the Virtual Machine

VirtualBox's **New VM Wizard** was used to create a Windows Server 2022 virtual machine named **DC02**.

| Setting | Value |
|---------|-------|
| VM Name | DC02 |
| VM Folder | `C:\Users\jeetr\VirtualBox VMs` |
| ISO Image | `C:\Users\jeetr\Downloads\SERVER_EVAL_x64FRE_en-us.iso` |
| OS Edition | Windows Server 2022 Standard Evaluation (64-bit) |
| RAM | 4096 MB |
| Disk | 50 GB dynamic VDI |

![VirtualBox New VM Wizard — DC02 configured](screenshots/01-vm-setup-dc02.png)

---

### Step 2 — Install Windows Server 2022

After starting the VM, the Windows Server setup launched. **Install now** was selected.

![Windows Server 2022 setup — Install now](screenshots/02-windows-setup-install-now.png)

Edition selected: **Windows Server 2022 Standard Evaluation (Desktop Experience)** for the full GUI. A custom install was chosen and the 50 GB virtual disk was selected as the target.

![Windows Server installation in progress](screenshots/03-windows-installing.png)

Installation took approximately 15 minutes. After reboot, a local Administrator password was set and the server was logged into for the first time.

---

### Step 3 — First Boot: SConfig

On first sign-in, **SConfig** launched automatically. At this point the server was still in workgroup mode (`WORKGROUP`) with a default auto-generated computer name (`WIN-CTK4HA3HJST`).

SConfig was exited to proceed with GUI-based configuration via Server Manager.

![SConfig on first boot — workgroup mode, computer name not yet set](screenshots/04-sconfig-first-boot.png)

---

### Step 4 — Launch Server Manager and Install the AD DS Role

Server Manager launched automatically after exiting SConfig. A Windows Admin Center promotional popup appeared and was dismissed.

![Server Manager Dashboard — first launch](screenshots/05-server-manager-dashboard.png)

From **Manage > Add Roles and Features**, the following were selected for installation:

- Active Directory Domain Services
- Group Policy Management
- Remote Server Administration Tools (automatically included)
  - Active Directory module for Windows PowerShell
  - AD DS Tools
  - Active Directory Administrative Center
  - AD DS Snap-Ins and Command-Line Tools

![Add Roles and Features — Confirmation page](screenshots/06-adds-role-confirmation.png)

After clicking **Install**, all components installed successfully.

![AD DS role installation complete](screenshots/07-adds-installation-complete.png)

---

### Step 5 — Promote the Server to a Domain Controller

With AD DS installed, the next step was promoting the server to a Domain Controller.

**PowerShell attempt and troubleshooting:**

A PowerShell command was attempted first:

```powershell
Install-ADDSForest -DomainName "lab.local" -InstallDns -SafeModeAdministratorPassword (ConvertTo-SecureString "Admin@1234" -AsPlainText -Force) - Force
```

This returned: `A positional parameter cannot be found that accepts argument '-'.`

The cause was a stray space before `-Force`, making PowerShell parse `- Force` as a positional argument instead of a switch. Rather than fix and retry, the GUI wizard was used instead.

![PowerShell Install-ADDSForest command error](screenshots/08-powershell-forest-error.png)

**GUI promotion via AD DS Configuration Wizard:**

The yellow notification flag in Server Manager was clicked and **Promote this server to a domain controller** was selected.

Settings used:

| Setting | Value |
|---------|-------|
| Deployment operation | Add a new forest |
| Root domain name | `lab.local` |
| Forest functional level | Windows Server 2016 |
| Domain functional level | Windows Server 2016 |
| DNS server | Checked (installed automatically) |
| DSRM password | Set |

**DNS delegation warning:**

During the DNS Options step, this warning appeared:

> A delegation for this DNS server cannot be created because the authoritative parent zone cannot be found...

This is expected in an isolated home lab. The domain `lab.local` has no parent zone in any real DNS hierarchy, so no delegation can be created. The warning was dismissed and the wizard continued.

![DNS delegation warning during DC promotion](screenshots/09-dns-delegation-warning.png)

After the wizard completed, the server rebooted automatically and came back as a Domain Controller.

![Administrator login after DC promotion reboot](screenshots/10-administrator-login-post-promotion.png)

---

### Step 6 — Confirm Domain Controller Promotion

After logging back in, **Server Manager** showed the AD DS and DNS roles installed with green status.

![Server Manager — AD DS and DNS roles confirmed](screenshots/11-server-manager-post-promotion.png)

The **Tools** menu now displayed all Active Directory management tools, confirming the promotion was successful:

- Active Directory Users and Computers
- Active Directory Administrative Center
- Active Directory Domains and Trusts
- Active Directory Sites and Services
- Active Directory Module for Windows PowerShell
- Group Policy Management
- ADSI Edit

![Server Manager Tools menu — all AD tools available](screenshots/12-server-manager-tools-menu.png)

---

### Step 7 — Create Organizational Units

**Active Directory Users and Computers (ADUC)** was opened from the Tools menu. The domain `lab.local` was visible in the left panel.

![ADUC — lab.local domain connected](screenshots/13-aduc-lab-local-domain.png)

Three OUs were created by right-clicking `lab.local` > New > Organizational Unit:

| OU Name | Distinguished Name |
|---------|--------------------|
| IT | `OU=IT,DC=lab,DC=local` |
| HR | `OU=HR,DC=lab,DC=local` |
| Finance | `OU=Finance,DC=lab,DC=local` |

All three OUs are visible in the left panel under `lab.local`.

![ADUC — IT, HR, Finance OUs created](screenshots/14-aduc-ou-structure.png)

---

### Step 8 — Create User Accounts

Five domain user accounts were created, each placed in the appropriate OU. Every account was configured with:

- Display name and User Principal Name in `user@lab.local` format
- Initial password set
- **"User must change password at next logon"** checked

![New Object — User: password dialog with "User must change password at next logon" checked](screenshots/15-aduc-new-user-password.png)

**Users created:**

| Username | Full Name | OU | Role |
|----------|-----------|----|------|
| jsmith | John Smith | IT | IT Technician |
| adavis | Amanda Davis | IT | Systems Administrator |
| rthomas | Rachel Thomas | HR | HR Coordinator |
| mklein | Mark Klein | Finance | Finance Analyst |
| lpatel | Lisa Patel | Finance | Accounts Payable |

**Finance OU** with Mark Klein and Lisa Patel:

![ADUC — Finance OU showing Mark Klein and Lisa Patel](screenshots/16-aduc-finance-ou-users.png)

---

### Step 9 — Create Security Groups

Three security groups were created, one per department. Each group was created by right-clicking the target OU > New > Group, then members were added via the **Members** tab in the group's Properties dialog.

| Group Name | OU | Scope | Type | Members |
|------------|----|-------|------|---------|
| IT-Team | IT | Global | Security | jsmith, adavis |
| HR-Team | HR | Global | Security | rthomas |
| Finance-Team | Finance | Global | Security | mklein, lpatel |

**IT-Team** with John Smith (jsmith) and Amanda Davis (adavis) as members:

![IT-Team group — Members tab showing jsmith and adavis](screenshots/17-aduc-it-team-members.png)

**HR-Team** with Rachel Thomas (rthomas) as member:

![HR-Team group — Members tab showing rthomas](screenshots/18-aduc-hr-team-members.png)

**Finance-Team** with Mark Klein (mklein) and Lisa Patel (lpatel) as members:

![Finance-Team group — Members tab showing mklein and lpatel](screenshots/19-aduc-finance-team-members.png)

---

### Step 10 — Create and Apply the IT-Security-Policy GPO

**Group Policy Management Console (GPMC)** was opened from the Tools menu.

A new GPO named **IT-Security-Policy** was created by right-clicking `lab.local` > Create a GPO in this domain and link it here. Linking at the domain level means the policy applies to all computers and users in `lab.local`.

![GPMC — IT-Security-Policy linked and enabled at lab.local](screenshots/20-gpmc-it-security-policy-linked.png)

The GPO was opened in the **Group Policy Management Editor** and configured with three policy areas.

![Group Policy Management Editor — IT-Security-Policy tree structure](screenshots/21-gpo-editor-security-settings-tree.png)

---

**Password Policy**

Path: `Computer Configuration > Windows Settings > Security Settings > Account Policies > Password Policy`

Minimum password length was set to **10 characters**:

![Password Policy — Minimum password length: 10 characters](screenshots/22-gpo-password-length-10-chars.png)

Password complexity requirements were set to **Enabled**:

![Password Policy — Password must meet complexity requirements: Enabled](screenshots/23-gpo-password-complexity-enabled.png)

---

**Account Lockout Policy**

Path: `Computer Configuration > Windows Settings > Security Settings > Account Policies > Account Lockout Policy`

| Setting | Value |
|---------|-------|
| Account lockout threshold | 3 invalid logon attempts |
| Account lockout duration | 30 minutes |
| Reset account lockout counter after | 30 minutes |

![Account Lockout Policy — threshold 3 attempts, duration 30 min, reset 30 min](screenshots/24-gpo-account-lockout-policy.png)

---

**USB / Removable Storage Block**

Path: `Computer Configuration > Administrative Templates > System > Removable Storage Access`

The **All Removable Storage classes: Deny all access** setting was set to **Enabled**, blocking all USB and removable media at the policy level.

![Removable Storage — Deny all access properties: Enabled](screenshots/25-gpo-usb-deny-all-access-enabled.png)

![Removable Storage Access list — Deny all access confirmed as Enabled](screenshots/26-gpo-removable-storage-list.png)

---

**Applying the GPO**

After saving all GPO settings, the following command was run on DC02 to apply the policy immediately without waiting for the default refresh interval:

```
gpupdate /force
```

Both Computer Policy and User Policy completed successfully:

![gpupdate /force — Computer Policy and User Policy both updated successfully](screenshots/27-gpupdate-force-success.png)

---

## Domain Structure

```
lab.local
├── OU: IT
│   ├── User: jsmith (John Smith) — IT Technician
│   ├── User: adavis (Amanda Davis) — Systems Administrator
│   └── Group: IT-Team [Global, Security]
├── OU: HR
│   ├── User: rthomas (Rachel Thomas) — HR Coordinator
│   └── Group: HR-Team [Global, Security]
└── OU: Finance
    ├── User: mklein (Mark Klein) — Finance Analyst
    ├── User: lpatel (Lisa Patel) — Accounts Payable
    └── Group: Finance-Team [Global, Security]

GPO: IT-Security-Policy (linked at lab.local, domain-wide)
    Password: min 10 characters, complexity required
    Lockout: 3 attempts, 30 min duration, reset after 30 min
    Storage: All removable storage access denied
```

---

## Skills Demonstrated

| Skill | Where Applied |
|-------|--------------|
| Deploy Windows Server 2022 from ISO in VirtualBox | Step 1-2 |
| Install and configure the AD DS role via Server Manager | Step 4 |
| Promote a standalone server to a Domain Controller | Step 5 |
| Troubleshoot a PowerShell syntax error in real time | Step 5 |
| Interpret and dismiss an expected DNS delegation warning | Step 5 |
| Navigate Active Directory Users and Computers (ADUC) | Steps 7-9 |
| Design and create an OU hierarchy aligned to business departments | Step 7 |
| Create domain user accounts with UPN format and password policy | Step 8 |
| Create Global Security groups and assign department members | Step 9 |
| Create and link a GPO at the domain level via GPMC | Step 10 |
| Configure password policy and account lockout policy in a GPO | Step 10 |
| Block removable storage access via Group Policy | Step 10 |
| Force immediate policy application with `gpupdate /force` | Step 10 |

---

## Relevance to IT Roles

| Role | How This Lab Applies |
|------|---------------------|
| Help Desk | Create and reset user accounts, manage group membership, troubleshoot login and lockout issues |
| IT Support | Diagnose domain join failures, GPO application problems, and account lockout causes |
| Systems Administrator | Deploy AD from scratch, manage OU structure, create and link domain-wide GPOs |
| Security Analyst | Enforce password complexity, lockout policy, and removable storage controls through Group Policy |

---

## When This Was Built

Built on **September 7, 2026** as a full rebuild from scratch. The original VM's VHD was lost during a disk path change in VirtualBox. The rebuild covered every step from VM creation through GPO enforcement in a single session.

---

*Built by Jeet Rachalla | [GitHub](https://github.com/Jeetrachalla)*

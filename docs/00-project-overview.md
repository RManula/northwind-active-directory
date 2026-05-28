# Northwind Solutions AD Lab — Full Project Overview

This document lists every task completed across all six phases of the project,
from initial design through to security hardening and attack/detect simulation.
Written as a reference for what was built, configured, automated, and documented.

---

## Phase 1 — Company Design

- Designed a fictional 30-person managed IT services company (Northwind Solutions)
- Built an org chart with 6 departments: Executive, IT, Finance, HR, Sales, Operations
- Created employees.csv as the single source of truth for all automation scripts
- Defined and locked naming conventions:
    - Usernames: firstname.lastname
    - Computer names: NW-TYPE-##
    - Security groups: GG-Department-Permission
    - OUs: plain readable names with Users and Computers sub-OUs
- Designed the full OU structure including a _Disabled OU for offboarded accounts
- Planned the IP addressing scheme (10.0.2.0/24, static servers, DHCP workstations)
- Documented all decisions with reasoning in docs/01-company-design.md

---

## Phase 2 — Build the Foundation

- Built three VMs from scratch in Oracle VirtualBox:
    - NW-DC-01: Windows Server 2016, Domain Controller and DNS
    - NW-FS-01: Windows Server 2016, File Server
    - NW-WS-01: Windows 10, domain-joined workstation
- Created a VirtualBox NAT Network (10.0.2.0/24) for inter-VM communication
- Configured static IPs on both servers (10.0.2.10 and 10.0.2.20)
- Installed and configured VirtualBox Guest Additions on all VMs
- Installed the Active Directory Domain Services role on NW-DC-01
- Promoted NW-DC-01 to Domain Controller for northwind.local
- Configured AD-integrated DNS zone for northwind.local
- Verified AD and DNS were functioning correctly
- Domain-joined NW-FS-01 and NW-WS-01 to northwind.local
- Verified domain login worked from the Windows 10 workstation

---

## Phase 3 — Company Structure

- Built the full OU tree using PowerShell (New-OUStructure.ps1):
    - Root OU: Northwind
    - 6 department OUs each with Users and Computers sub-OUs
    - _Disabled OU with Users sub-OU for offboarded accounts
- Created all department security groups using PowerShell (New-SecurityGroups.ps1):
    - Read/write and read-only groups per department
    - GG-AllStaff group for company-wide policies
    - GG-IT-Admins for elevated permissions
- Bulk-created all 30 user accounts from employees.csv using PowerShell (New-BulkUsers.ps1):
    - Usernames generated automatically as firstname.lastname
    - Accounts placed in correct department OUs
    - Added to correct security groups based on department
    - Passwords set with must-change-at-first-login enforced
- Verified all users, OUs, and group memberships in Active Directory Users and Computers

---

## Phase 4 — Policies and Access

- Configured Group Policy Objects (GPOs) for security and access control:
    - Default Domain Policy: password complexity, minimum length, lockout threshold
    - Department GPOs: desktop restrictions, mapped drives, software policies
    - IT department GPO: admin tool access
- Set up shared folders on NW-FS-01:
    - One share per department (Finance, HR, Sales, Operations, IT, Executive)
    - Company-wide shared drive accessible to GG-AllStaff
- Configured NTFS permissions on all shares:
    - Department groups assigned read/write to their own share
    - Other departments blocked
    - IT Admins granted full control across all shares
- Mapped network drives via GPO so users get their department drive automatically at login
- Tested permissions by logging in as users from different departments
- Documented all share paths, permissions, and GPO settings

---

## Phase 5 — Help Desk Scenarios

Each scenario is written as a real ticket with steps taken to resolve it.

- Password reset for a locked-out user
- Account unlock after too many failed login attempts
- New starter onboarding: create account, add to groups, set up drive access
- User offboarding: disable account, move to _Disabled OU, remove group memberships,
  revoke drive access — automated with Disable-OffboardedUser.ps1
- User changes department: update OU, update group memberships, update drive access
- Workstation not seeing the domain: DNS troubleshooting and re-join
- User cannot access a shared folder: permissions investigation and fix
- Password expiry report: identify users whose passwords expire in the next 14 days
  using Get-PasswordExpiry.ps1
- Stale account detection: flag accounts inactive for 90+ days
  using Get-StaleAccounts.ps1
- Full AD audit report: export all user accounts, status, groups, and last logon
  to HTML and CSV using Get-ADAuditReport.ps1

---

## Phase 6 — Security Hardening and Attack/Detect

### Hardening

- Renamed the default built-in Administrator account
- Disabled the Guest account
- Configured fine-grained password policies for privileged accounts
- Enabled and configured audit policies:
    - Logon and logoff events
    - Account management (creation, deletion, modification)
    - Privilege use
    - Object access (file share access)
- Restricted RDP access to IT staff only via GPO
- Disabled unnecessary services and roles on the DC
- Enabled SMB signing to prevent man-in-the-middle attacks
- Reviewed and documented the principle of least privilege across all accounts
- Verified no standard users had local admin rights on workstations

### Attack Simulation

- Performed a password spray attack against domain accounts
- Demonstrated Kerberoasting — extracted and attempted to crack a service ticket
- Simulated an enumeration attack using AD queries to map the domain
- Demonstrated pass-the-hash using a compromised local admin hash

### Detection and Analysis

- Pulled Windows Security Event Logs and identified attack indicators:
    - Event ID 4625: failed logon attempts (password spray pattern)
    - Event ID 4769: Kerberos service ticket requests (Kerberoasting indicator)
    - Event ID 4624/4672: successful logon with special privileges
    - Event ID 4732/4728: user added to privileged group
- Documented what each attack looks like in the logs
- Wrote a detection summary explaining how a SOC analyst would spot each technique

---

## Skills Demonstrated

### Active Directory Administration
- Domain design and deployment from scratch
- OU structure, delegation, and GPO targeting
- User and group lifecycle management
- DNS configuration and troubleshooting

### PowerShell Automation
- Bulk user creation from CSV
- OU and group provisioning scripts
- Offboarding automation
- Audit and reporting scripts

### Group Policy
- Password and lockout policies
- Drive mapping via GPO
- Desktop restrictions
- Scoped policies per department

### File Server and Permissions
- SMB share creation and management
- NTFS permission design and implementation
- Access control testing

### Help Desk Operations
- Documented ticket-based scenarios
- Account management, access issues, onboarding, offboarding
- Real troubleshooting steps recorded

### Security
- Hardening a Windows Server environment
- Understanding and simulating common AD attack techniques
- Reading and interpreting Windows Security Event Logs
- Connecting attacker behaviour to log evidence

---

*All infrastructure is virtualised. All company data is fictional.*

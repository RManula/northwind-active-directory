# Northwind Solutions — Active Directory Lab

A simulated enterprise Active Directory environment built in VirtualBox. Covers the full lifecycle of identity and access management for a fictional 30-person managed IT services company — domain design, bulk provisioning, Group Policy, file permissions, help desk operations, and security hardening with attack/detect simulation.

## Environment

| Component | Detail |
|---|---|
| Domain | northwind.local |
| Domain Controller | NW-DC-01 — Windows Server 2016 |
| File Server | NW-FS-01 — Windows Server 2016 |
| Workstation | NW-WS-01 — Windows 10 Pro |
| Network | 10.0.2.0/24 — VirtualBox NAT Network |
| Host | Windows 10, Oracle VirtualBox |

## Progress

| Phase | Description | Status |
|---|---|---|
| 1 | Company design — org chart, naming conventions, OU structure, IP plan | Complete |
| 2 | Foundation — DC deployment, DNS, NW-FS-01 and NW-WS-01 domain joins | In Progress |
| 3 | Company structure — OUs, security groups, 30 bulk-provisioned users | Not Started |
| 4 | Policies and access — GPOs, file shares, NTFS permissions | Not Started |
| 5 | Help desk scenarios — 11 documented ticket resolutions | Not Started |
| 6 | Security hardening + attack/detect simulation | Not Started |

## What This Covers

**Active Directory Administration**
- Domain design and deployment from scratch on Windows Server 2016
- OU structure that mirrors the org chart, designed for department-level GPO targeting
- AD-integrated DNS — zone configuration, SRV records, forwarder setup, verification
- User and group lifecycle — bulk provisioning from CSV through to offboarding automation
- FSMO roles, Global Catalog, Kerberos and LDAP service record verification

**PowerShell Automation**
- Bulk user provisioning from `employees.csv` — usernames, OU placement, group membership
- OU tree and security group provisioning scripts
- Offboarding automation — disable, reset password, move to `_Disabled` OU, strip group memberships
- Audit scripts — stale accounts, password expiry, full AD report to HTML and CSV

**Group Policy**
- Password policy and account lockout GPOs
- Department-scoped network drive mapping
- Desktop restrictions and screensaver lockout
- RDP access restricted to IT staff only via GPO

**File Server and Permissions**
- SMB shares per department on a dedicated file server (separate VM from the DC — deliberate security boundary)
- NTFS permissions applied via security groups, not individual accounts
- Cross-department access testing — confirm least-privilege is enforced

**Help Desk Operations**
- 11 documented scenarios written as tickets: account lockouts, password resets, onboarding, offboarding, department transfers, DNS troubleshooting, permission requests, stale account audits
- Each scenario includes the problem, steps taken, and resolution

**Security Hardening and Attack/Detect**
- Before/after hardening comparison — default settings documented first
- Attack simulation: password spray, Kerberoasting, AD enumeration
- Windows Security Event Log analysis — Event IDs 4625, 4740, 4769, 4624/4672, 4732/4728
- SOC-level write-up connecting each attack technique to its log evidence

## Org Chart

![Northwind Org Chart](screenshots/01-design/Org%20Chart.jpg)

---
*Portfolio project — all company data is fictional.*

# Northwind Solutions — Active Directory Lab

A fully simulated enterprise Active Directory environment built in
VirtualBox, documenting the complete lifecycle of identity management,
group policy, file permissions, help desk operations, and security
hardening for a fictional 30-person managed IT services company.

## What This Project Covers
- Active Directory design, deployment and administration
- PowerShell automation — bulk provisioning, auditing, offboarding
- Group Policy Objects for department-level access control
- File server setup with NTFS permission management
- Help desk scenario documentation — real ticket resolutions
- Security hardening and attack/detect demonstration

## Environment
| Component | Detail |
|---|---|
| Domain | northwind.local |
| Domain Controller | Windows Server 2016 — NW-DC-01 |
| File Server | Windows Server 2016 — NW-FS-01 |
| Workstation | Windows 10 — NW-WS-01 |
| Network | 10.0.2.0/24 (VirtualBox NAT Network) |
| Hypervisor | Oracle VirtualBox |

## Project Structure
| Phase | Topic | Status |
|---|---|---|
| 1 | Company design — org chart, naming, OUs, IP plan | 🔄 In progress |
| 2 | Foundation — DC, DNS, domain join | ⏳ Not started |
| 3 | Company structure — OUs, groups, bulk users | ⏳ Not started |
| 4 | Policies and access — GPOs, file shares | ⏳ Not started |
| 5 | Help desk scenarios | ⏳ Not started |
| 6 | Security hardening + attack/detect | ⏳ Not started |

## Org Chart
![Northwind Org Chart](screenshots/01-design/Org_Chart.png)

---
*Built as a portfolio project. All company data is fictional.*

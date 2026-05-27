# Northwind Solutions — Active Directory Lab

A simulated enterprise Active Directory environment built in VirtualBox. The project documents the full lifecycle of identity management, group policy, file permissions, help desk operations, and security hardening for a fictional 30-person managed IT services company.

## What This Project Covers
- Active Directory design, deployment and administration
- PowerShell automation — bulk provisioning, auditing, offboarding
- Group Policy Objects for department-level access control
- File server setup with NTFS permission management
- Help desk scenario documentation
- Security hardening and attack/detect lab

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
| 1 | Company design — org chart, naming, OUs, IP plan | In Progress |
| 2 | Foundation — DC, DNS, domain join | Not Started |
| 3 | Company structure — OUs, groups, bulk users | Not Started |
| 4 | Policies and access — GPOs, file shares | Not Started |
| 5 | Help desk scenarios | Not Started |
| 6 | Security hardening + attack/detect | Not Started |

## Org Chart
![Northwind Org Chart](screenshots/01-design/Org%20Chart.jpg)

---
*Built as a portfolio project. All company data is fictional.*

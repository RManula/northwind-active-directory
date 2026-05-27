# CLAUDE.md — Northwind Solutions AD Project

## What This Project Is
A fully simulated Active Directory environment for a fictional managed IT
services company called Northwind Solutions, based in Brisbane.
Built in VirtualBox on a Windows host (20 GB RAM).
Goal: professional GitHub portfolio project targeting help desk and junior
sysadmin roles, with a full security hardening and attack/detect layer for
a cybersecurity career track.

---

## The Company
| Detail | Value |
|---|---|
| Name | Northwind Solutions |
| Industry | Managed IT Services |
| Location | Brisbane, Australia |
| Domain | northwind.local |
| Staff | 30 employees across 6 departments |
| Departments | Executive, IT, Finance, HR, Sales, Operations |

---

## VM Plan
| VM | OS | Role | IP |
|---|---|---|---|
| NW-DC-01 | Windows Server 2016 | Domain Controller + DNS | 10.0.2.10 static |
| NW-FS-01 | Windows Server 2016 | File Server | 10.0.2.20 static |
| NW-WS-01 | Windows 10 | Workstation, domain joined | DHCP |

---

## Folder Structure
```
northwind-active-directory/
├── CLAUDE.md
├── README.md
├── .gitignore
├── data/
│   └── employees.csv
├── docs/
│   ├── 01-company-design.md
│   ├── 02-foundation-setup.md
│   ├── 03-company-structure.md
│   ├── 04-policies-and-access.md
│   ├── 05-helpdesk-scenarios.md
│   └── 06-security-hardening.md
├── scripts/
│   ├── New-OUStructure.ps1
│   ├── New-BulkUsers.ps1
│   ├── New-SecurityGroups.ps1
│   ├── Get-ADAuditReport.ps1
│   ├── Get-StaleAccounts.ps1
│   ├── Get-PasswordExpiry.ps1
│   └── Disable-OffboardedUser.ps1
└── screenshots/
    ├── 01-design/
    ├── 02-foundation/
    ├── 03-structure/
    ├── 04-policies/
    ├── 05-helpdesk/
    └── 06-security/
```

---

## Six Phases
| Phase | Name | Status |
|---|---|---|
| 1 | Company design | 🔄 In progress |
| 2 | Build the foundation | ⏳ Not started |
| 3 | Company structure | ⏳ Not started |
| 4 | Policies and access | ⏳ Not started |
| 5 | Help desk scenarios | ⏳ Not started |
| 6 | Security hardening + attack/detect | ⏳ Not started |

---

## Phase 1 — Design Artifacts
| Artifact | Status |
|---|---|
| 1. Org chart + employees.csv | ✅ Complete |
| 2. Naming conventions | ✅ Complete |
| 3. OU structure | ✅ Complete |
| 4. IP addressing plan | ✅ Complete |

---

## Naming Conventions

- **Usernames:** `firstname.lastname` lowercase — example `margaret.chen` — collision adds number suffix (e.g. `margaret.chen2`)
- **Computer names:** `NW-TYPE-##` — examples `NW-DC-01`, `NW-FS-01`, `NW-WS-01`
- **Security groups:** `GG-Department-Permission` — examples `GG-Finance-RW`, `GG-IT-Admins`, `GG-AllStaff`
- **OUs:** plain readable names — root OU is `Northwind`, each department has `Users` and `Computers` sub-OUs, `_Disabled` OU for offboarded accounts

**OU Structure:** Root OU is `Northwind`. Six department OUs — `Executive`, `IT`, `Finance`, `HR`, `Sales`, `Operations` — each with `Users` and `Computers` sub-OUs. One special OU called `_Disabled` with a `Users` sub-OU for offboarded accounts. No Groups sub-OU at this size.

**IP Plan:** Network `10.0.2.0/24`. `NW-DC-01` static `10.0.2.10`, `NW-FS-01` static `10.0.2.20`, workstations on DHCP. DNS points to `10.0.2.10`.

---

## Locked Decisions
- **Automation:** Heavy PowerShell. Scripts read employees.csv. Every script documented with WHY, not just WHAT.
- **Security:** Full hardening + attack/detect in Phase 6. This is a cyber project, not just an IT admin lab.
- **Documentation:** Every task framed as a real business scenario. Decisions documented with reasoning.
- **OU design:** Tree mirrors the org chart. Each department has Users and Computers sub-OUs only. No Groups sub-OU — deliberate decision for a 30-person company.
- **File server:** Separate VM from DC — deliberate security decision, DC should run minimal roles.
- **Commits:** One clean commit per completed artifact. Meaningful messages only.

---

## Working Rules
- One file at a time. Never build Phase N+1 before Phase N is documented and committed.
- Always explain WHY a decision was made, not just WHAT was done.
- Security is considered in every task — never an afterthought.
- Commit messages must be meaningful.

---

## What To Do Next
> **Current task: Write `docs/01-company-design.md`**
> All four Phase 1 artifacts are locked. Next task is to write the complete Phase 1 design document.
> Do not start Phase 2 until this file is written and committed.

---

*This file is updated after every major decision. Always read it before doing anything.*

# Northwind Solutions — Active Directory Lab

A fully simulated Active Directory environment for a fictional managed IT services company, built as a professional portfolio project. Designed to demonstrate real-world help desk and junior sysadmin skills, with a full security hardening and attack/detect layer for a cybersecurity career track.

---

## The Company

**Northwind Solutions** is a fictional managed IT services provider based in Brisbane, Australia, with 30 employees across six departments: Executive, IT, Finance, HR, Sales, and Operations.

| Detail | Value |
|---|---|
| Domain | northwind.local |
| Staff | 30 employees |
| Departments | Executive, IT, Finance, HR, Sales, Operations |

---

## Lab Environment

Built in VirtualBox on a Windows host (20 GB RAM).

| VM | OS | Role | IP |
|---|---|---|---|
| NW-DC-01 | Windows Server 2016 | Domain Controller + DNS | 10.0.2.10 (static) |
| NW-FS-01 | Windows Server 2016 | File Server | 10.0.2.20 (static) |
| NW-WS-01 | Windows 10 | Domain-joined Workstation | DHCP |

> The file server runs on a separate VM from the DC by design — a Domain Controller should run minimal roles to reduce its attack surface.

---

## Project Phases

| Phase | Name | Status |
|---|---|---|
| 1 | Company design | 🔄 In progress |
| 2 | Build the foundation | ⏳ Not started |
| 3 | Company structure | ⏳ Not started |
| 4 | Policies and access | ⏳ Not started |
| 5 | Help desk scenarios | ⏳ Not started |
| 6 | Security hardening + attack/detect | ⏳ Not started |

---

## Repository Structure

```
northwind-active-directory/
├── data/               # Source data (employees.csv)
├── docs/               # Phase documentation — decisions with reasoning
├── scripts/            # PowerShell automation scripts
└── screenshots/        # Evidence screenshots, organised by phase
```

All PowerShell scripts are documented with **why** decisions were made, not just what the script does. Every task is framed as a real business scenario.

---

## Skills Demonstrated

- Active Directory design and administration
- PowerShell automation (bulk user creation, auditing, offboarding)
- Group Policy configuration and security hardening
- Help desk scenario walkthroughs
- Attack simulation and detection (Phase 6)
- Documentation written for a professional IT audience

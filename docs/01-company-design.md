# Phase 1 — Company Design

Status: Complete

---

## Overview

This document covers the design decisions made before any infrastructure was built. All four artifacts below were locked before a single VM was provisioned or AD object created — naming conventions, OU structure and IP plan are agreed here and don't change later without a documented reason.

1. Org chart and employee roster
2. Naming conventions
3. OU structure
4. IP addressing plan

---

## The Company

**Northwind Solutions** is a managed IT services provider based in Brisbane, Australia, with 30 employees across six departments.

| Detail | Value |
|---|---|
| Domain | northwind.local |
| Staff | 30 employees |
| Departments | Executive, IT, Finance, HR, Sales, Operations |

The `.local` suffix keeps the domain fully internal and non-routable. It's the standard choice for an isolated lab, though in a production environment the mDNS conflict issue with `.local` would be worth discussing before committing to it.

---

## Artifact 1 — Org Chart and Employee Roster

### Department Breakdown

| Department | Head Count |
|---|---|
| Executive | 3 |
| IT | 5 |
| Finance | 5 |
| HR | 4 |
| Sales | 7 |
| Operations | 6 |
| **Total** | **30** |

### Reporting Structure

The CEO has two C-level direct reports — CFO and COO. Department managers sit one level below that. The hierarchy is clean enough that department boundaries map directly to OUs without any awkward exceptions.

```
Margaret Chen — CEO
├── David Okafor — CFO
│   └── Robert Kim — Finance Manager
│       ├── Emma Watson — Senior Accountant
│       ├── Hassan Ali — Accountant
│       ├── Grace Liu — Payroll Officer
│       └── Daniel Foster — Accounts Clerk
└── Priya Sharma — COO
    ├── James Sullivan — IT Manager
    │   ├── Aisha Rahman — Systems Administrator
    │   ├── Tom Nguyen — Help Desk Technician
    │   ├── Lucas Brandt — Help Desk Technician
    │   └── Sofia Reyes — Network Engineer
    ├── Karen Mitchell — HR Manager
    │   ├── Olivia Bennett — HR Officer
    │   ├── Marcus Webb — Recruitment Specialist
    │   └── Fatima Noor — HR Assistant
    ├── Andrew Scott — Sales Manager
    │   ├── Jessica Taylor — Senior Sales Rep
    │   ├── Ben Carter — Sales Rep
    │   ├── Mia Hernandez — Sales Rep
    │   ├── Ryan Cooper — Sales Rep
    │   ├── Chloe Adams — Sales Rep
    │   └── Nathan Price — Sales Coordinator
    └── Victor Osei — Operations Manager
        ├── Hannah Schmidt — Project Coordinator
        ├── Ethan Walker — Service Delivery Lead
        ├── Zara Khan — Logistics Officer
        ├── Liam Murphy — Operations Analyst
        └── Isabella Rossi — Facilities Coordinator
```

### Employee Data

All employee records are in `data/employees.csv` with columns `FirstName`, `LastName`, `Department`, `JobTitle`, `Manager`. The Phase 3 scripts read from this file directly for user creation, group membership and OU placement, so employee data stays in one place rather than being scattered across scripts.

---

## Artifact 2 — Naming Conventions

These were locked before any AD objects were created. Renaming objects after deployment causes group membership and policy issues, so it's worth agreeing on a standard upfront.

### Usernames

Format: `firstname.lastname`, all lowercase.

Examples: `margaret.chen`, `james.sullivan`, `tom.nguyen`

If two employees share a name, the second account gets a number suffix — `john.smith2`. The first account is never renamed.

`firstname.lastname` is readable and predictable. Users can usually work out their own username without calling IT. Shorter formats like `jsmith` get messy when multiple staff share a surname initial.

### Computer Names

Format: `NW-TYPE-##`

| Type Code | Meaning |
|---|---|
| `DC` | Domain Controller |
| `FS` | File Server |
| `WS` | Workstation |
| `LT` | Laptop |

Examples: `NW-DC-01`, `NW-FS-01`, `NW-WS-01`

The name encodes the company prefix, machine role and sequence number. When you're looking at a list of computer objects in AD — or reading an event log — you know exactly what each machine is without having to open it.

### Security Groups

Format: `GG-Department-Permission`

- `GG` = Global Group (the appropriate scope for departmental access control within a single domain)
- `Department` = department name or functional label
- `Permission` = what the group grants (`RW`, `RO`, `Admins`, `AllStaff`, etc.)

| Group Name | Purpose |
|---|---|
| `GG-Finance-RW` | Read/write access to Finance file share |
| `GG-Finance-RO` | Read-only access to Finance share |
| `GG-IT-Admins` | Elevated permissions for IT staff |
| `GG-AllStaff` | All 30 employees — used for company-wide policies |
| `GG-HR-RW` | Read/write access to HR file share |

The `GG-` prefix makes security groups easy to identify in AD and in PowerShell output, and keeps them grouped together alphabetically in the console.

### OU Names

Plain readable names — no codes, no underscores except for `_Disabled`. An IT admin browsing ADUC should be able to navigate without needing a reference document. The underscore on `_Disabled` puts it at the top of the OU list so offboarded accounts stay visible.

---

## Artifact 3 — OU Structure

### Design

The OU tree follows the org chart. Each department has its own OU with `Users` and `Computers` sub-OUs, which allows GPOs to target users or workstations in a specific department without affecting others. A policy locking down Finance workstations links to `Finance > Computers` only.

All company objects sit under a single root OU (`Northwind`) rather than directly under the domain root. GPOs linked at the domain root apply to Domain Controllers as well, so keeping company objects under their own OU avoids accidentally pushing desktop policies to the DC. It also makes the structure easier to manage if the domain is ever extended or merged.

There's no Groups sub-OU. With 30 staff, security groups are manageable without a dedicated container, and adding one just means extra navigation. That can be reconsidered if headcount grows significantly.

### OU Tree

```
DC=northwind,DC=local
└── OU=Northwind
    ├── OU=Executive
    │   ├── OU=Users
    │   └── OU=Computers
    ├── OU=IT
    │   ├── OU=Users
    │   └── OU=Computers
    ├── OU=Finance
    │   ├── OU=Users
    │   └── OU=Computers
    ├── OU=HR
    │   ├── OU=Users
    │   └── OU=Computers
    ├── OU=Sales
    │   ├── OU=Users
    │   └── OU=Computers
    ├── OU=Operations
    │   ├── OU=Users
    │   └── OU=Computers
    └── OU=_Disabled
        └── OU=Users
```

### Offboarded Accounts

When someone leaves, their account is disabled and moved to `_Disabled > Users` rather than deleted straight away. Deleting an AD account removes the SID, which leaves gaps in event logs — historical entries that referenced that account lose their context. A disabled account keeps the SID intact. It also means the account is accessible if something needs to be checked after the person has left. Accounts in `_Disabled` are reviewed and removed after 90 days.

---

## Artifact 4 — IP Addressing Plan

### Network

| Detail | Value |
|---|---|
| Network | `10.0.2.0/24` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `10.0.2.1` (VirtualBox NAT Network) |
| DNS Server | `10.0.2.10` (NW-DC-01) |

A `/24` gives 254 usable addresses, which is more than enough for this environment with room to add machines during the lab.

### Static Assignments

| Hostname | Role | IP Address |
|---|---|---|
| `NW-DC-01` | Domain Controller + DNS | `10.0.2.10` |
| `NW-FS-01` | File Server | `10.0.2.20` |

Both servers have static IPs. DNS records, GPO drive mappings and any firewall rules all reference these addresses — if a DHCP lease assigned a different IP, several things would break at the same time. Statics in the low end of the range (`10.0.2.10–.20`) keep them clear of the DHCP pool, which should be configured to start at `10.0.2.100` or above.

### DNS

DNS runs on NW-DC-01 as an AD-integrated zone for `northwind.local`. Domain-joined machines register themselves dynamically, and the DC publishes the SRV records clients need to locate Kerberos and LDAP services. External queries are forwarded to the VirtualBox NAT gateway.

Running DNS on the DC is standard at this scale — a separate DNS server would add infrastructure overhead with no practical benefit for 30 users.

---

## Summary

| Artifact | Status |
|---|---|
| 1. Org chart + employees.csv | Complete |
| 2. Naming conventions | Complete |
| 3. OU structure | Complete |
| 4. IP addressing plan | Complete |

Phase 2 covers building the foundation — provisioning NW-DC-01, installing AD DS, configuring DNS and domain-joining NW-WS-01.

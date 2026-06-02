# Phase 1 - Company Design

All design decisions locked before any infrastructure was built. Naming conventions, OU structure and IP plan agreed here so nothing needs renaming later.

---

## The Company

**Northwind Solutions** - managed IT services provider, Brisbane, 30 employees across six departments.

| Detail | Value |
|---|---|
| Domain | northwind.local |
| Staff | 30 |
| Departments | Executive, IT, Finance, HR, Sales, Operations |

`.local` keeps the domain internal and non-routable, standard for an isolated lab.

---

## 1. Org Chart and Employee Roster

| Department | Headcount |
|---|---|
| Executive | 3 |
| IT | 5 |
| Finance | 5 |
| HR | 4 |
| Sales | 7 |
| Operations | 6 |
| **Total** | **30** |

```
Margaret Chen (CEO)
├── David Okafor (CFO)
│   └── Robert Kim (Finance Manager)
│       ├── Emma Watson (Senior Accountant)
│       ├── Hassan Ali (Accountant)
│       ├── Grace Liu (Payroll Officer)
│       └── Daniel Foster (Accounts Clerk)
└── Priya Sharma (COO)
    ├── James Sullivan (IT Manager)
    │   ├── Aisha Rahman (Systems Administrator)
    │   ├── Tom Nguyen (Help Desk Technician)
    │   ├── Lucas Brandt (Help Desk Technician)
    │   └── Sofia Reyes (Network Engineer)
    ├── Karen Mitchell (HR Manager)
    │   ├── Olivia Bennett (HR Officer)
    │   ├── Marcus Webb (Recruitment Specialist)
    │   └── Fatima Noor (HR Assistant)
    ├── Andrew Scott (Sales Manager)
    │   ├── Jessica Taylor (Senior Sales Rep)
    │   ├── Ben Carter (Sales Rep)
    │   ├── Mia Hernandez (Sales Rep)
    │   ├── Ryan Cooper (Sales Rep)
    │   ├── Chloe Adams (Sales Rep)
    │   └── Nathan Price (Sales Coordinator)
    └── Victor Osei (Operations Manager)
        ├── Hannah Schmidt (Project Coordinator)
        ├── Ethan Walker (Service Delivery Lead)
        ├── Zara Khan (Logistics Officer)
        ├── Liam Murphy (Operations Analyst)
        └── Isabella Rossi (Facilities Coordinator)
```

All 30 records are in `data/employees.csv`. The provisioning scripts in Phase 3 read from that file directly.

---

## 2. Naming Conventions

### Usernames

Format: `firstname.lastname` (lowercase)

Examples: `margaret.chen`, `tom.nguyen`

If two staff share a name, the second account gets a number suffix (`john.smith2`). First account is never renamed.

### Computer Names

Format: `NW-TYPE-##`

| Code | Role |
|---|---|
| DC | Domain Controller |
| FS | File Server |
| WS | Workstation |
| LT | Laptop |

Examples: `NW-DC-01`, `NW-FS-01`, `NW-WS-01`

Name encodes company, role and sequence number. Useful when reading event logs or browsing AD.

### Security Groups

Format: `GG-Department-Permission`

| Group | Purpose |
|---|---|
| GG-Finance-RW | Read/write to Finance share |
| GG-Finance-RO | Read-only to Finance share |
| GG-IT-Admins | Elevated permissions for IT staff |
| GG-AllStaff | All 30 users, used for company-wide policies |
| GG-HR-RW | Read/write to HR share |

`GG-` prefix keeps security groups identifiable in ADUC and PowerShell output.

### OUs

Plain readable names. No codes. The `_Disabled` OU uses an underscore so it sorts to the top of the list in ADUC - makes offboarded accounts easy to find.

---

## 3. OU Structure

Tree follows the org chart. Each department gets `Users` and `Computers` sub-OUs so GPOs can target users or machines in a department independently.

All company objects sit under a root `Northwind` OU rather than directly under the domain root. GPOs linked at the domain root hit Domain Controllers too, which is not what you want for desktop policies.

No Groups sub-OU at this size. Not worth the extra navigation for 30 staff.

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

Offboarded accounts go to `_Disabled > Users` rather than being deleted. Deleting an AD account removes the SID which leaves gaps in event logs. Disabled accounts stay in place for 90 days then get removed.

---

## 4. IP Addressing Plan

| Host | Role | IP |
|---|---|---|
| NW-DC-01 | Domain Controller + DNS | 10.0.2.10 |
| NW-FS-01 | File Server | 10.0.2.20 |
| NW-WS-01 | Workstation | DHCP |

Network: `10.0.2.0/24`, gateway `10.0.2.1` (VirtualBox NAT Network).

Both servers are static. DHCP pool starts at 10.0.2.100 to keep statics and dynamic leases separated.

DNS runs on NW-DC-01 as an AD-integrated zone. External queries forwarded to 8.8.8.8.

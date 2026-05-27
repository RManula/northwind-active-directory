# Phase 1 — Company Design

**Status:** ✅ Complete  
**Purpose:** Establish all design decisions before touching any infrastructure. Nothing is built until it is documented and reasoned here.

---

## Overview

Before a single VM is provisioned or an AD object created, the environment needs a design. This document locks in the four foundational artifacts for Northwind Solutions' Active Directory deployment:

1. Org chart and employee roster
2. Naming conventions
3. OU structure
4. IP addressing plan

Every decision in this document exists because a real sysadmin would need to make the same call. The reasoning is documented alongside the decision — because in a real environment, the person who comes after you needs to understand *why* things were set up the way they were, not just *what* was done.

---

## The Company

**Northwind Solutions** is a managed IT services provider based in Brisbane, Australia. The company has 30 employees across six departments.

| Detail | Value |
|---|---|
| Domain | northwind.local |
| Staff | 30 employees |
| Departments | Executive, IT, Finance, HR, Sales, Operations |

The `.local` domain suffix is used because this is a fully internal, non-internet-routable domain. In a production environment this decision would warrant more discussion (`.local` has known mDNS conflicts), but for an isolated lab it is the standard choice.

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

The company has a flat-ish hierarchy. The CEO (Margaret Chen) has two direct reports at C-level: CFO and COO. All department managers report to either the CFO or COO. This structure directly influences OU and group policy design — department boundaries are clean, which makes delegation straightforward.

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

### Source File

Full employee data is stored in `data/employees.csv` with the columns: `FirstName`, `LastName`, `Department`, `JobTitle`, `Manager`. This CSV is the single source of truth for all bulk PowerShell operations in Phase 3 — user creation, group membership, and OU placement all read from this file.

**Why a CSV?** It separates data from logic. The PowerShell scripts that create AD objects do not have any employee data hardcoded into them. If a new employee is added or a department changes, you update the CSV and re-run the script — you do not touch the script itself.

---

## Artifact 2 — Naming Conventions

Naming conventions are locked before any AD object is created. Changing a naming convention after deployment means renaming objects across the directory, which causes group membership and policy issues. Get it right before you start.

### Usernames

**Format:** `firstname.lastname` — all lowercase

**Examples:** `margaret.chen`, `james.sullivan`, `tom.nguyen`

**Collision handling:** If two employees share the same first and last name, the second account gets a number suffix — `john.smith2`. The first account is never renamed.

**Why this format?** It is readable, predictable, and consistent with what most corporate environments use. Users can guess their own username. IT can find an account without running a search. Shorter formats like `jsmith` become ambiguous at scale and create more collisions — `firstname.lastname` scales cleanly to several hundred users before collision handling is ever needed.

### Computer Names

**Format:** `NW-TYPE-##`

| Type Code | Meaning |
|---|---|
| `DC` | Domain Controller |
| `FS` | File Server |
| `WS` | Workstation |
| `LT` | Laptop |

**Examples:** `NW-DC-01`, `NW-FS-01`, `NW-WS-01`, `NW-WS-02`

**Why this format?** The name tells you the company (`NW`), the role of the machine (`DC`, `WS`), and its sequence number. You can read a computer object in AD and immediately know what it is and where it should be, without opening it. This matters in event logs, in SIEM tooling, and when you are troubleshooting at 2am.

### Security Groups

**Format:** `GG-Department-Permission`

- `GG` = Global Group (the AD group scope, chosen for departmental access control within a single domain)
- `Department` = the department name, or a functional label
- `Permission` = what the group grants (e.g. `RW`, `RO`, `Admins`, `AllStaff`)

**Examples:**

| Group Name | Purpose |
|---|---|
| `GG-Finance-RW` | Finance department — read/write to Finance file share |
| `GG-Finance-RO` | Read-only access to Finance share (e.g. for auditors) |
| `GG-IT-Admins` | IT staff with elevated permissions |
| `GG-AllStaff` | All 30 employees — used for company-wide policies |
| `GG-HR-RW` | HR department — read/write to HR file share |

**Why the `GG-` prefix?** It makes groups instantly identifiable in AD and in PowerShell output. When you are looking at a user's group memberships, you can tell at a glance which entries are departmental access groups versus other group types. It also keeps all security groups sorted together alphabetically in the AD console.

### OU Names

**Format:** Plain readable English names — no codes, no underscores (except for the special `_Disabled` OU)

**Why plain names?** OUs are navigational — an IT manager browsing ADUC should be able to find `Finance > Users` without a decoder ring. Cryptic codes add no value in OU names. The complexity lives in the group names and GPOs, not the OU tree.

The `_Disabled` OU uses an underscore prefix deliberately — it sorts to the top of the OU list in ADUC, making offboarded accounts immediately visible and easy to audit.

---

## Artifact 3 — OU Structure

### Design Principles

The OU tree mirrors the org chart. Each department gets its own OU with `Users` and `Computers` sub-OUs. This matters because Group Policy is applied at the OU level — a policy that locks down workstations in Finance should not accidentally apply to Sales workstations. Keeping users and computers in separate sub-OUs gives fine-grained GPO targeting.

**No Groups sub-OU** — at 30 employees, all security groups are managed centrally. Adding a Groups sub-OU at this size creates extra navigation overhead with no operational benefit. This decision is revisited if the company grows past ~100 employees.

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

### Why a Root `Northwind` OU?

All company objects live under a single root OU rather than directly under the domain root. This is a deliberate security and management decision:

- GPOs applied at the domain root affect every object including Domain Controllers. Keeping company objects under their own root OU means you can apply company-wide policies (e.g. password policy, screen lock) without risking DC misconfiguration.
- If Northwind Solutions were ever acquired or merged into a larger domain, the entire company's OU tree can be moved or delegated as a single unit.
- It clearly separates company-managed objects from built-in AD containers like `CN=Users` and `CN=Computers`.

### `_Disabled` OU

When an employee leaves the company, their account is not deleted immediately — it is disabled and moved to `_Disabled > Users`. This is standard practice for several reasons:

- **Audit trail:** Deleted AD accounts leave gaps in event logs. A disabled account retains its SID and all historical log references remain valid.
- **Data recovery:** If a user's mailbox, files, or permissions need to be reviewed post-departure, the account is still accessible.
- **Compliance:** Many industries require that access records are retained for a defined period. Disabling rather than deleting supports this.

Accounts in `_Disabled` are reviewed quarterly and permanently deleted after the retention period (typically 90 days).

---

## Artifact 4 — IP Addressing Plan

### Network

| Detail | Value |
|---|---|
| Network | `10.0.2.0/24` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `10.0.2.1` (VirtualBox NAT Network) |
| DNS Server | `10.0.2.10` (NW-DC-01) |

The `/24` subnet gives 254 usable host addresses — more than sufficient for a 30-person company with room for growth and lab expansion.

### Static Assignments

| Hostname | Role | IP Address | Reason for Static |
|---|---|---|---|
| `NW-DC-01` | Domain Controller + DNS | `10.0.2.10` | DNS server — must never change. Clients point to this IP. |
| `NW-FS-01` | File Server | `10.0.2.20` | File share mappings reference this IP. A DHCP change breaks mapped drives. |

**Why static IPs for servers?** DHCP is designed for devices that come and go. Servers are infrastructure — their addresses are referenced in DNS records, GPO drive mappings, firewall rules, and monitoring configs. A DHCP lease renewal that assigns a different IP breaks all of those at once. Static assignments eliminate that failure mode entirely.

### DHCP Range

Workstations and any other devices pick up addresses from DHCP. In this lab, VirtualBox's built-in DHCP service handles assignment within the `10.0.2.0/24` range. Static assignments are kept in the low range (`10.0.2.10–.20`) to avoid conflicts — DHCP should be configured to start its pool at `10.0.2.100` or higher.

### DNS

All DNS queries resolve through `NW-DC-01` at `10.0.2.10`. The Domain Controller runs the AD-integrated DNS zone for `northwind.local`. This means:

- Domain-joined machines can resolve `northwind.local` hostnames automatically via DNS dynamic update.
- The DC registers its own SRV records so clients can locate domain services (Kerberos, LDAP) without manual configuration.
- `NW-DC-01` forwards external DNS queries (anything not in `northwind.local`) to an upstream resolver — in this lab, the VirtualBox NAT gateway handles internet resolution.

**Why not run DNS on a separate server?** For a domain this size, co-locating DNS on the DC is standard and appropriate. Separating them adds infrastructure complexity for no security or resilience benefit at 30 users. This is revisited if a second DC is added.

---

## Phase 1 Sign-Off

All four artifacts are locked. Nothing in this section changes without a documented reason and a new commit.

| Artifact | Status |
|---|---|
| 1. Org chart + employees.csv | ✅ Locked |
| 2. Naming conventions | ✅ Locked |
| 3. OU structure | ✅ Locked |
| 4. IP addressing plan | ✅ Locked |

**Next phase:** Phase 2 — Build the Foundation. Provision NW-DC-01, install AD DS, configure DNS, and domain-join NW-WS-01. See `02-foundation-setup.md` when that phase begins.

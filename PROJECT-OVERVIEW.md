# Northwind Solutions — Project Overview and Task List
# Read this file before starting any task in any session.
# Status: ✅ Done | 🔄 In Progress | ⏳ Not Started
# Update statuses as tasks are completed.
================================================================

## What This Project Is

A fully simulated enterprise Active Directory environment for a fictional
30-person managed IT services company called Northwind Solutions, based in
Brisbane. Built in VirtualBox on a Windows host with 20 GB RAM.

Goal: professional GitHub portfolio targeting help desk and junior sysadmin
roles, with a full security hardening and attack/detect layer signalling a
cybersecurity career track.

Domain:   northwind.local
Network:  10.0.2.0/24 (VirtualBox NAT Network)

VMs:
  NW-DC-01   Windows Server 2016   Domain Controller + DNS   10.0.2.10 static
  NW-FS-01   Windows Server 2016   File Server               10.0.2.20 static
  NW-WS-01   Windows 10            Workstation               DHCP

Key files to read before doing anything:
  CLAUDE.md              — project rules, locked decisions, naming conventions
  PROJECT-OVERVIEW.md    — this file, full task list and status
  data/employees.csv     — 30 staff, input for all provisioning scripts

================================================================
## PHASE 1 — Company Design
================================================================
✅  Design fictional company — Northwind Solutions, Brisbane, 30 staff
✅  Build org chart — 6 departments, full hierarchy
✅  Create data/employees.csv — single source of truth for all scripts
✅  Lock naming conventions:
        Usernames:       firstname.lastname (e.g. margaret.chen)
        Computer names:  NW-TYPE-## (e.g. NW-DC-01)
        Security groups: GG-Department-Permission (e.g. GG-Finance-RW)
        OUs:             Plain readable names, Users + Computers sub-OUs
✅  Design OU structure — mirrors org chart, includes _Disabled OU
✅  Design IP addressing plan — static servers, DHCP workstations
✅  Write docs/01-company-design.md
✅  Push Phase 1 to GitHub


================================================================
## PHASE 2 — Build the Foundation
================================================================
✅  Create NW-DC-01 VM in VirtualBox
✅  Install Windows Server 2016 Desktop Experience on NW-DC-01
✅  Rename server to NW-DC-01
✅  Install VirtualBox Guest Additions on NW-DC-01
✅  Create NAT Network in VirtualBox (10.0.2.0/24)
✅  Attach NW-DC-01 to NAT Network
✅  Set static IP 10.0.2.10 on NW-DC-01
✅  Install AD DS role on NW-DC-01
✅  Promote NW-DC-01 to Domain Controller for northwind.local
✅  Configure AD-integrated DNS zone for northwind.local
✅  Verify AD and DNS are functioning correctly
✅  Create NW-FS-01 VM — Windows Server 2016, 40 GB, 2 GB RAM
✅  Install Windows Server 2016 Desktop Experience on NW-FS-01
✅  Rename to NW-FS-01
✅  Set static IP 10.0.2.20 on NW-FS-01
✅  Attach NW-FS-01 to NAT Network
✅  Install Guest Additions on NW-FS-01
✅  Domain join NW-FS-01 to northwind.local
⏳  Create NW-WS-01 VM — Windows 10, 40 GB, 2 GB RAM
⏳  Install Windows 10 on NW-WS-01
⏳  Rename to NW-WS-01
⏳  Set DNS to point to 10.0.2.10 on NW-WS-01
⏳  Attach NW-WS-01 to NAT Network
⏳  Install Guest Additions on NW-WS-01
⏳  Domain join NW-WS-01 to northwind.local
⏳  Verify domain login works from NW-WS-01
⏳  Write docs/02-foundation-setup.md
⏳  Push Phase 2 to GitHub


================================================================
## PHASE 3 — Company Structure
================================================================
⏳  Write and run New-OUStructure.ps1 — builds entire OU tree:
        Northwind > Executive, IT, Finance, HR, Sales, Operations
        Each with Users and Computers sub-OUs
        Plus _Disabled > Users for offboarded accounts
⏳  Verify OU tree in ADUC matches Phase 1 design exactly
⏳  Write and run New-SecurityGroups.ps1 — creates all GG- groups:
        GG-Finance-RW, GG-Finance-RO
        GG-HR-RW, GG-HR-RO
        GG-Sales-RW, GG-Operations-RW
        GG-IT-Admins, GG-AllStaff
⏳  Verify all groups created and visible in ADUC
⏳  Write and run New-BulkUsers.ps1 — reads employees.csv, creates all 30 users:
        Usernames generated as firstname.lastname automatically
        Accounts placed in correct department OUs
        Added to correct security groups by department
        Passwords set with must-change-at-first-login enforced
⏳  Verify all 30 users in correct OUs
⏳  Verify all group memberships are correct
⏳  Test login with a domain user account from NW-WS-01
⏳  Write docs/03-company-structure.md
⏳  Push Phase 3 to GitHub


================================================================
## PHASE 4 — Policies and Access
================================================================

### Group Policy Objects
⏳  Create GPO — Password Policy:
        Minimum 10 characters, complexity required, 90 day expiry
⏳  Create GPO — Account Lockout Policy:
        5 failed attempts, 30 minute lockout duration
⏳  Create GPO — Desktop Restrictions for standard users
⏳  Create GPO — Screensaver timeout with password lock
⏳  Create GPO — Mapped drives per department (auto at login)
⏳  Create GPO — Restrict IT admin tools to IT department only
⏳  Link all GPOs to correct OUs
⏳  Test GPOs applying correctly from NW-WS-01
⏳  Run Get-ADAuditReport.ps1 — verify policy application

### File Server and Shares
⏳  Create shared folders on NW-FS-01 — one per department plus AllStaff
⏳  Configure NTFS permissions per department security group:
        Department RW group — read/write to own share
        All other departments — no access
        GG-IT-Admins — full control across all shares
        GG-AllStaff — read/write to AllStaff share only
⏳  Map network drives via GPO — users get department drive at login
⏳  Test: Finance user can access Finance share
⏳  Test: Finance user cannot access HR share
⏳  Test: IT admin can access all shares
⏳  Write docs/04-policies-and-access.md
⏳  Push Phase 4 to GitHub


================================================================
## PHASE 5 — Help Desk Scenarios
================================================================
# Each scenario documented as a real ticket with steps taken

⏳  Scenario 01 — New starter onboarding
        New Finance employee joins Northwind.
        Create account, add to correct OU and groups,
        confirm drive mapping, test login from NW-WS-01.

⏳  Scenario 02 — Account lockout
        User locked out after failed password attempts.
        Identify locked account in ADUC, unlock it,
        document resolution and cause.

⏳  Scenario 03 — Password reset
        User forgot password, cannot log in.
        Reset via ADUC, force change at next login,
        verify user can authenticate.

⏳  Scenario 04 — Permission request
        Sales user needs read access to Operations share.
        Verify request is legitimate, add to correct group,
        test access, document approval and change.

⏳  Scenario 05 — Wrong group membership
        User has access they should not have.
        Audit group memberships, remove incorrect entry,
        verify access revoked, document finding.

⏳  Scenario 06 — Department transfer
        Employee moves from Sales to Finance.
        Update OU placement, update group memberships,
        update drive mappings, verify new access works,
        verify old access is revoked.

⏳  Scenario 07 — Workstation cannot see the domain
        NW-WS-01 fails to find northwind.local.
        DNS troubleshooting steps, fix, re-join if needed,
        document root cause and resolution.

⏳  Scenario 08 — Employee offboarding
        Staff member leaves Northwind.
        Run Disable-OffboardedUser.ps1 — disables account,
        resets password, moves to _Disabled OU,
        removes group memberships. Document full process.

⏳  Scenario 09 — Password expiry warning
        Run Get-PasswordExpiry.ps1.
        Identify accounts expiring within 14 days.
        Document accounts found and action taken.

⏳  Scenario 10 — Stale account audit
        Run Get-StaleAccounts.ps1.
        Flag accounts inactive for 90+ days.
        Review each, disable or document exception.

⏳  Scenario 11 — Full AD audit report
        Run Get-ADAuditReport.ps1.
        Export all users, status, groups, last logon
        to HTML and CSV. Document findings.

⏳  Write docs/05-helpdesk-scenarios.md
⏳  Push Phase 5 to GitHub


================================================================
## PHASE 6 — Security Hardening and Attack/Detect
================================================================

### Step 1 — Audit Default Weak Settings (document before state)
⏳  Document default password policy — too short, no complexity
⏳  Document default account lockout — not configured
⏳  Document default audit policy — minimal logging enabled
⏳  Document share permissions — excessive access by default
⏳  Run Get-ADAuditReport.ps1 — capture baseline snapshot
⏳  Screenshot all default settings before touching anything

### Step 2 — Harden the Environment
⏳  Rename built-in Administrator account
⏳  Disable Guest account
⏳  Harden password policy — 12+ chars, complexity, 90 day expiry
⏳  Configure account lockout — 5 attempts, 30 min lockout
⏳  Configure fine-grained password policy for privileged accounts
⏳  Enable Windows Security Auditing via GPO:
        Logon and logoff events (success and failure)
        Account management (create, delete, modify)
        Privilege use
        Object access (file share access)
        Policy changes
⏳  Restrict RDP access to IT staff only via GPO
⏳  Restrict local admin rights on workstations
⏳  Disable unnecessary services on DC
⏳  Enable SMB signing to prevent man-in-the-middle attacks
⏳  Tighten share permissions — remove excessive access
⏳  Document every change with before/after comparison

### Step 3 — Attack Simulation
⏳  Simulate password spray — repeated failed logins against
        multiple domain accounts from NW-WS-01
⏳  Simulate Kerberoasting — request service ticket,
        attempt to extract and crack it
⏳  Simulate unauthorised share access — user attempts to
        access a department share they have no rights to
⏳  Simulate AD enumeration — query domain to map users,
        groups, and computers

### Step 4 — Detect the Attacks in Event Logs
⏳  Open Event Viewer on NW-DC-01
⏳  Find and screenshot Event ID 4625 — failed logon (spray pattern)
⏳  Find and screenshot Event ID 4740 — account locked out
⏳  Find and screenshot Event ID 4769 — Kerberos ticket request
⏳  Find and screenshot Event ID 4624/4672 — privileged logon
⏳  Find and screenshot Event ID 4732/4728 — group membership change
⏳  Document what each Event ID means and how a SOC analyst
        would use it to detect the attack

### Step 5 — Final Audit
⏳  Run Get-ADAuditReport.ps1 — post-hardening snapshot
⏳  Compare before and after — document every improvement
⏳  Run Get-StaleAccounts.ps1 — final clean sweep
⏳  Write docs/06-security-hardening.md
⏳  Push Phase 6 to GitHub


================================================================
## FINAL — Polish and Publish
================================================================
⏳  Update README.md — mark all phases complete with final statuses
⏳  Update README.md — add full skills demonstrated section
⏳  Final review of all six docs/ files — consistent formatting
⏳  Final review of all screenshots/ — named correctly per phase
⏳  Final review of all scripts/ — headers, comments, and reasoning
⏳  Update CLAUDE.md — mark project complete
⏳  Final commit: "docs: Northwind AD Lab — project complete"
⏳  Push everything to GitHub
⏳  Share repo link for final review


================================================================
## WHAT THIS DEMONSTRATES TO A RECRUITER
================================================================

Active Directory Administration
  - Domain design and deployment from scratch
  - OU structure design, implementation, and management
  - User and group lifecycle — creation through offboarding
  - Group Policy design, targeting, and verification
  - DNS configuration and troubleshooting

PowerShell Automation
  - Bulk user provisioning from CSV input
  - OU tree and security group provisioning scripts
  - Offboarding automation in a single command
  - Audit and reporting scripts (HTML and CSV output)
  - Stale account detection

Help Desk Operations
  - Documented ticket-based scenario resolutions
  - Account lockout, password reset, onboarding, offboarding
  - Permission management and access troubleshooting
  - Department transfers and group membership changes
  - DNS troubleshooting on a domain-joined workstation

Security and Hardening
  - Before/after hardening documentation
  - Windows Security Event Log analysis
  - Common AD attack technique simulation
  - Event ID identification and SOC-level interpretation
  - SMB signing, audit policy, least privilege

Professional Practices
  - Full design documentation before any build
  - Consistent naming conventions throughout
  - Git version control with meaningful commit messages
  - Structured GitHub portfolio with six phase documents
  - Every decision documented with reasoning not just steps

================================================================
# HOW TO USE THIS FILE
================================================================
- Read this file at the start of every session
- Update task statuses as work is completed (⏳ → 🔄 → ✅)
- If a task is blocked or changed, add a note next to it
- Cross-reference CLAUDE.md for locked decisions and naming rules
- Cross-reference employees.csv for user data
- Never start a new phase until the previous phase doc is
  written, committed, and pushed to GitHub
================================================================

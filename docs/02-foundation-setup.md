# Phase 2 — Foundation Setup

Status: In Progress

---

## Overview

This document covers building the three VMs that make up the Northwind Solutions lab environment and joining them into a working domain. Steps are recorded in the order they were completed, with configuration details and any decisions noted along the way.

| VM | Role | OS | IP |
|---|---|---|---|
| NW-DC-01 | Domain Controller + DNS | Windows Server 2016 | 10.0.2.10 (static) |
| NW-FS-01 | File Server | Windows Server 2016 | 10.0.2.20 (static) |
| NW-WS-01 | Workstation | Windows 10 | DHCP |

All three VMs run on a VirtualBox NAT Network (10.0.2.0/24). The NAT Network allows inter-VM communication while still providing internet access through the host.

---

## Step 1 — Create NW-DC-01 VM

**Status: Complete**

Created the NW-DC-01 virtual machine in VirtualBox with the following specs:

| Setting | Value |
|---|---|
| VM Name | NW-DC-01 |
| OS | Windows Server 2016 (64-bit) |
| RAM | 2048 MB |
| CPUs | 2 |
| Disk | 50 GB (VDI, dynamically allocated) |
| Network | NAT (temporary — changed to NAT Network in Step 5) |
| ISO | Windows Server 2016 Standard Evaluation |

The network adapter is left as NAT at creation time. VirtualBox 7.1 requires the NAT Network to be created separately (Step 4) before it can be assigned to a VM. The adapter is switched while the VM is powered off.

**Screenshots:** `01-vm-creation-start.png`, `02-vm-hardware-specs.png`, `03-vm-storage-settings.png`, `04-nwdc01-vm-created.png`

---

## Step 2 — Install Windows Server 2016

**Status: Complete**

Installed Windows Server 2016 Standard (Desktop Experience) on NW-DC-01. Desktop Experience selected to get the full GUI — the Server Core option provides command line only and is not suitable for this lab environment.

Installation type: Custom (clean install to the 50 GB virtual disk).
Administrator password set during installation.

**Screenshots:** `05-windows-installation.png`, `06-server2016-desktop-installed.png`, `07-ws2016-edition-select.png`

---

## Step 3 — Rename Server to NW-DC-01

**Status: Complete**

Server name confirmed as NW-DC-01 post-installation. Verified in System Properties.

**Screenshot:** verified in System Properties post-rename. No separate screenshot captured — rename was confirmed as part of initial setup.

---

## Step 4 — Create the NAT Network in VirtualBox

**Status: Complete**

Created a NAT Network in VirtualBox Network Manager with the following settings:

| Setting | Value |
|---|---|
| Name | Northwind-NAT |
| IPv4 Prefix | 10.0.2.0/24 |
| DHCP | Enabled |

NAT Network allows all VMs on the same network to communicate with each other, while still routing outbound traffic through the host for internet access. Standard NAT (the default) isolates each VM — they cannot see each other, which would prevent domain joining.

**Screenshot:** `08-nat-network-created.png`

---

## Step 5 — Attach NW-DC-01 to the NAT Network

**Status: Complete**

Changed NW-DC-01 network adapter from NAT to NAT Network (Northwind-NAT) while the VM was powered off. VM restarted after the change.

**Screenshot:** `09-nwdc01-adapter-nat-network.png`

---

## Step 6 — Assign Static IP 10.0.2.10

**Status: Complete**

Configured static IP on the Ethernet adapter inside NW-DC-01:

| Setting | Value |
|---|---|
| IP Address | 10.0.2.10 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 10.0.2.1 |
| Preferred DNS | 10.0.2.10 |
| Alternate DNS | 8.8.8.8 |

DNS is set to point to itself (10.0.2.10) because this server will become the DNS server for the domain after AD DS promotion. Static IP is required — DNS records, GPO drive mappings, and firewall rules all reference this address.

**Screenshots:** `10-static-ip-configured.png`, `11-ipconfig-verified.png`

---

## Step 7 — Install AD DS Role

**Status: Complete**

Installed the Active Directory Domain Services role via Server Manager (Add Roles and Features). No additional features were required beyond those automatically selected by the role installation.

**Screenshot:** `12-adds-role-installed.png`

---

## Step 8 — Promote to Domain Controller

**Status: Complete**

Promoted NW-DC-01 to Domain Controller for a new forest. Configuration:

| Setting | Value |
|---|---|
| Deployment | New forest |
| Root domain | northwind.local |
| Forest functional level | Windows Server 2016 |
| Domain functional level | Windows Server 2016 |
| DNS Server | Installed during promotion |
| NetBIOS name | NORTHWIND |

Server rebooted automatically after promotion. Logged back in as NORTHWIND\Administrator.

**Screenshots:** `13-dc-promotion-config.png`, `14-promotion-complete-afterwithadministratorloginview.png`

---

## Step 9 — Configure DNS Zone for northwind.local

**Status: Complete**

The forward lookup zone for `northwind.local` was created automatically during DC promotion in Step 8 — AD DS installs DNS and creates an AD-integrated zone as part of the promotion process. AD-integrated means zone data is stored in Active Directory itself rather than in a flat file, which means it replicates automatically to any additional DCs added later.

Two verification commands confirmed DNS is working correctly:

```
nslookup northwind.local       → returned 10.0.2.10 (the DC)
nslookup nw-dc-01.northwind.local  → returned 10.0.2.10
```

The second lookup initially failed because the DC had not yet written its own A record into the zone. Running `ipconfig /registerdns` forced the DC to register the record. This is a normal first-boot behaviour — DNS registration happens on a timer and `ipconfig /registerdns` triggers it immediately.

These two lookups confirm the DNS foundation is solid before any other VMs are brought up. When NW-FS-01 and NW-WS-01 are configured to use `10.0.2.10` as their DNS server and attempt to domain-join, the first thing they do is query DNS for `northwind.local` to locate the DC. If either lookup had failed here, domain joining would fail too.

**DNS forwarder configured:** `8.8.8.8` added as a forwarder so the DC can resolve external names (internet DNS) while still being authoritative for `northwind.local`.

**Screenshot:** `15-dns-manager-zone-verified.png`

---

## Step 10 — Verify AD and DNS

**Status: Complete**

Four verifications were run from an elevated PowerShell session on NW-DC-01. All passed.

---

### Verification 1 — Get-ADDomain

```powershell
Get-ADDomain
```

This cmdlet queries Active Directory and returns the properties of the domain itself. It confirms that AD DS is running and responding to queries. The key fields in the output are:

- **DNSRoot: northwind.local** — the domain exists in DNS as configured
- **Name: NORTHWIND** — the NetBIOS name assigned during promotion
- **DomainMode** — confirms the functional level is Windows Server 2016, meaning all DC-level features from that version are available
- **InfrastructureMaster, PDCEmulator, RIDMaster** — these are FSMO roles (Flexible Single Master Operations). They are special domain-wide functions that can only be performed by one DC at a time. As the first and only DC in the forest, NW-DC-01 holds all five FSMO roles automatically. They will matter later when understanding how AD handles things like password changes (PDC Emulator) and new object creation (RID Master).

If this command returned nothing or threw an error, it would mean AD DS was not running or the domain was not configured, and no further work could continue.

---

### Verification 2 — Get-ADDomainController

```powershell
Get-ADDomainController
```

This returns properties of the DC itself rather than the domain. Key fields:

- **Name: NW-DC-01** — the computer name matches the naming convention locked in Phase 1
- **IPv4Address: 10.0.2.10** — the static IP assigned in Step 6 is correctly registered
- **IsGlobalCatalog: True** — the first DC in a forest is automatically a Global Catalog server. The Global Catalog holds a partial copy of every object in the forest and is required for user authentication and forest-wide searches. If this were False, logins would fail in a multi-domain forest scenario.

This confirms the machine is registered as a Domain Controller, not just a member server with AD DS installed.

---

### Verification 3 — dcdiag /test:dns

```powershell
dcdiag /test:dns
```

DCDiag (Domain Controller Diagnostics) is a built-in Windows Server tool that runs a battery of health tests against the DC. The `/test:dns` flag runs specifically the DNS tests, which check:

- The DNS service is running and accessible
- The forward lookup zone for `northwind.local` exists
- The DC has registered the required **SRV records** in DNS — records like `_ldap._tcp.northwind.local` and `_kerberos._tcp.northwind.local` that client machines query to locate LDAP and Kerberos services on the domain. Without these records, domain joining works but logins may fail silently.
- All DNS records are consistent and correct

Passing this test confirms DNS is not just doing basic name resolution but is fully ready to support domain operations. A test failure here would have needed to be resolved before building any other VMs.

---

### Verification 4 — net share

```powershell
net share
```

Lists all shared resources on the server. The two critical ones confirmed here are:

- **SYSVOL** — a shared folder that stores Group Policy templates, scripts, and other domain-wide data. It is replicated between Domain Controllers and must exist for Group Policy to function. Every domain-joined computer accesses SYSVOL when it starts up to check for policies. Created automatically during DC promotion.
- **NETLOGON** — the share that domain-joined computers contact during logon to retrieve login scripts and verify credentials. Also created automatically during promotion.

If either share was missing, domain joining of NW-FS-01 and NW-WS-01 would either fail or produce intermittent authentication errors. Their presence here confirms the DC is fully operational.

---

**Screenshots:** `16-Get-ADDomain&Get-ADDomainController_Output.png`, `17-dcdiag-dns-passed.png`, `18-net_share_result.png`

---

## Step 11 — Create NW-FS-01 VM, Install Server 2016

**Status: Complete**

Created the NW-FS-01 virtual machine in VirtualBox and installed Windows Server 2016 Standard (Desktop Experience).

| Setting | Value |
|---|---|
| VM Name | NW-FS-01 |
| OS | Windows Server 2016 (64-bit) |
| RAM | 2048 MB |
| Disk | 40 GB (VDI, dynamically allocated) |
| Network | NAT (temporary — changed to NAT Network in Step 12) |

NW-FS-01 is intentionally kept as a separate VM from NW-DC-01 rather than running the file server role on the DC. A Domain Controller should run minimal roles — adding a file server to a DC means that a compromised file share could give an attacker direct access to AD. Keeping them separate is a basic security boundary.

The 40 GB disk is smaller than the DC's 50 GB because this VM does not need to store AD database files or system state backups. The extra space on the DC is there to accommodate those.

**Screenshots:** `19-nwfs01-vm-created.png`, `20-nwfs01-server2016-installed.png`

---

## Step 12 — Configure NW-FS-01 (Rename, Network, Static IP)

**Status: Complete**

Three configuration changes made to NW-FS-01 before domain joining:

**Renamed to NW-FS-01** via Server Manager > Local Server > Computer Name. At this point the machine is still a Workgroup member — domain joining is a separate step. The rename just ensures the correct name is registered when it joins the domain.

**Attached to NAT Network** — adapter changed from NAT to Northwind-NAT in VirtualBox settings while the VM was powered off. Same NAT Network NW-DC-01 is on, which allows the two VMs to communicate directly.

**Static IP configured:**

| Setting | Value |
|---|---|
| IP Address | 10.0.2.20 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 10.0.2.1 |
| Preferred DNS | 10.0.2.10 |

DNS points to NW-DC-01 at `10.0.2.10`. This is what allows NW-FS-01 to resolve `northwind.local` when it attempts to domain join in Step 13. Without DNS pointing at the DC, the domain join wizard cannot find the domain and will fail.

`ipconfig /all` confirmed all settings applied correctly. `Primary DNS Suffix` was blank at this stage — this populates with `northwind.local` automatically after domain joining.

**Screenshots:** `21-nwfs01-adapter-nat-network.png`, `22-nwfs01-renamed.png`, `23-nwfs01-static-ip-configured.png`, `24-nwfs01-ipconfig-verified.png`

---

## Step 13 — Domain Join NW-FS-01

**Status: Complete**

NW-FS-01 was joined to `northwind.local` using the PowerShell method:

```powershell
Add-Computer -DomainName northwind.local -Credential NORTHWIND\Administrator -Restart
```

The command prompts for the domain Administrator password, performs the join, and reboots automatically. After reboot, logged back in as `NORTHWIND\Administrator` using the Other User option on the login screen.

Domain join confirmed in two places:
- **Server Manager > Local Server** — Domain field shows `northwind.local`
- **NW-DC-01 ADUC > Computers container** — NW-FS-01 appears as a computer object, confirming the DC registered the join centrally

The file server is now a domain member. All domain user accounts will be able to authenticate on NW-FS-01, and Group Policy from the DC will apply to it. File share permissions configured in Phase 4 will reference domain security groups rather than local accounts.

**Screenshots:** `25-nwfs01-domain-join-success.png`, `26-nwfs01-domain-login.png`, `27-nwfs01-domain-joined-confirmed.png`

---

## Step 14 — Create NW-WS-01 VM, Install Windows 10

**Status: Not Started**

---

## Step 15 — Configure NW-WS-01 (Rename, Network, DNS)

**Status: Not Started**

---

## Step 16 — Domain Join NW-WS-01

**Status: Not Started**

---

## Step 17 — Verify Domain Login from NW-WS-01

**Status: Not Started**

---

## Summary

| Step | Description | Status |
|---|---|---|
| 1 | Create NW-DC-01 VM | Complete |
| 2 | Install Windows Server 2016 | Complete |
| 3 | Rename server to NW-DC-01 | Complete |
| 4 | Create NAT Network in VirtualBox | Complete |
| 5 | Attach NW-DC-01 to NAT Network | Complete |
| 6 | Assign static IP 10.0.2.10 | Complete |
| 7 | Install AD DS role | Complete |
| 8 | Promote to Domain Controller | Complete |
| 9 | Configure DNS zone for northwind.local | Complete |
| 10 | Verify AD and DNS | Complete |
| 11 | Create NW-FS-01 VM, install Server 2016 | Complete |
| 12 | Configure NW-FS-01 | Complete |
| 13 | Domain join NW-FS-01 | Complete |
| 14 | Create NW-WS-01 VM, install Windows 10 | Not Started |
| 15 | Configure NW-WS-01 | Not Started |
| 16 | Domain join NW-WS-01 | Not Started |
| 17 | Verify domain login from NW-WS-01 | Not Started |

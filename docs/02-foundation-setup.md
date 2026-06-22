# Phase 2 - Foundation Setup

Building the three VMs and joining them into a working domain.

| VM | Role | OS | IP |
|---|---|---|---|
| NW-DC-01 | Domain Controller + DNS | Windows Server 2016 | 10.0.2.10 |
| NW-FS-01 | File Server | Windows Server 2016 | 10.0.2.20 |
| NW-WS-01 | Workstation | Windows 10 | DHCP |

All three run on a VirtualBox NAT Network (10.0.2.0/24) so they can reach each other.

---

## Step 1 - Create NW-DC-01 VM

| Setting | Value |
|---|---|
| RAM | 2048 MB |
| CPUs | 2 |
| Disk | 50 GB, VDI, dynamically allocated |
| Network | NAT for now, switched to NAT Network in step 4 |

![VM creation](../screenshots/02-foundation/01-vm-creation-start.png)
![Hardware specs](../screenshots/02-foundation/02-vm-hardware-specs.png)
![Storage settings](../screenshots/02-foundation/03-vm-storage-settings.png)
![VM ready](../screenshots/02-foundation/04-nwdc01-vm-created.png)

---

## Step 2 - Install Windows Server 2016

Desktop Experience (full GUI). Clean install to the 50 GB disk.

![Installation](../screenshots/02-foundation/05-windows-installation.png)
![Edition select](../screenshots/02-foundation/07-ws2016-edition-select.png)
![Desktop up](../screenshots/02-foundation/06-server2016-desktop-installed.png)

---

## Step 3 - Rename to NW-DC-01

Done via System Properties before anything else. Server comes up with a random generated name by default.

---

## Step 4 - Create the NAT Network

Created Northwind-NAT (10.0.2.0/24) in VirtualBox Network Manager with DHCP on. Standard NAT isolates each VM so they cannot see each other - NAT Network fixes that.

![NAT Network created](../screenshots/02-foundation/08-nat-network-created.png)

---

## Step 5 - Attach NW-DC-01 to Northwind-NAT

Changed adapter from NAT to Northwind-NAT with the VM powered off.

![Adapter changed](../screenshots/02-foundation/09-nwdc01-adapter-nat-network.png)

---

## Step 6 - Static IP 10.0.2.10

| Setting | Value |
|---|---|
| IP | 10.0.2.10 |
| Subnet | 255.255.255.0 |
| Gateway | 10.0.2.1 |
| DNS | 10.0.2.10 (itself) |
| Alt DNS | 8.8.8.8 |

DNS points to itself because this box becomes the DNS server after promotion. Static IP so nothing references a moving target.

![IP configured](../screenshots/02-foundation/10-static-ip-configured.png)
![ipconfig verified](../screenshots/02-foundation/11-ipconfig-verified.png)

---

## Step 7 - Install AD DS Role

Added via Server Manager. No extra features needed beyond what the wizard auto-selects.

![AD DS installed](../screenshots/02-foundation/12--adds-role-installed.png)

---

## Step 8 - Promote to Domain Controller

New forest. Reboots automatically, comes back as NORTHWIND\Administrator.

| Setting | Value |
|---|---|
| Forest | New |
| Domain | northwind.local |
| Forest/Domain level | Windows Server 2016 |
| DNS | Installed during promotion |
| NetBIOS | NORTHWIND |

![Promotion config](../screenshots/02-foundation/13-dc-promotion-config.png)
![Promotion complete](../screenshots/02-foundation/14-promotion-complete-afterwithadministratorloginview.png)

---

## Step 9 - Configure DNS

The northwind.local forward lookup zone gets created automatically during promotion as an AD-integrated zone. Added 8.8.8.8 as a forwarder for external lookups.

Had to run `ipconfig /registerdns` to get the DC to register its own A record - it was not auto-registering on first boot. Both lookups worked after that.

```
nslookup northwind.local           -> 10.0.2.10
nslookup nw-dc-01.northwind.local  -> 10.0.2.10
```

![DNS verified](../screenshots/02-foundation/15-dns-manager-zone-verified.png)

---

## Step 10 - Verify AD and DNS

Four checks before building the other VMs.

**Get-ADDomain** - AD is up, FSMO roles confirmed. All five sit on NW-DC-01 as the first DC in the forest.

**Get-ADDomainController** - NW-DC-01 registered as a DC at 10.0.2.10, Global Catalog enabled.

**dcdiag /test:dns** - SRV records for _ldap and _kerberos are registered. These are what clients query to locate the DC when joining the domain. All passed.

**net share** - NETLOGON and SYSVOL shares present. Both are required for domain joining and Group Policy.

![Get-ADDomain and Get-ADDomainController](../screenshots/02-foundation/16-Get-ADDomain&Get-ADDomainController_Output.png)
![dcdiag passed](../screenshots/02-foundation/17-dcdiag-dns-passed.png)
![net share](../screenshots/02-foundation/18-net_share_result.png)

---

## Step 11 - Create NW-FS-01 VM

| Setting | Value |
|---|---|
| RAM | 2048 MB |
| Disk | 40 GB, VDI, dynamically allocated |
| Network | NAT for now |

File server is on a separate VM from the DC. Running shares on a DC is bad practice - a compromised share would mean direct access to AD.

![NW-FS-01 created](../screenshots/02-foundation/19-nwfs01-vm-created.png)
![Server 2016 installed](../screenshots/02-foundation/20-nwfs01-server2016-installed.png)

---

## Step 12 - Configure NW-FS-01

Powered off, switched adapter to Northwind-NAT, booted back up. Renamed to NW-FS-01 via Server Manager. Set static IP.

| Setting | Value |
|---|---|
| IP | 10.0.2.20 |
| Subnet | 255.255.255.0 |
| Gateway | 10.0.2.1 |
| DNS | 10.0.2.10 |

![Adapter changed](../screenshots/02-foundation/21-nwfs01-adapter-nat-network.png)
![Renamed](../screenshots/02-foundation/22-nwfs01-renamed.png)
![IP configured](../screenshots/02-foundation/23-nwfs01-static-ip-configured.png)
![ipconfig verified](../screenshots/02-foundation/24-nwfs01-ipconfig-verified.png)

---

## Step 13 - Domain Join NW-FS-01

```powershell
Add-Computer -DomainName northwind.local -Credential NORTHWIND\Administrator -Restart
```

Came back up as NORTHWIND\Administrator. Server Manager shows Domain: northwind.local. NW-FS-01 shows up in ADUC under Computers on the DC.

![Domain join](../screenshots/02-foundation/25-nwfs01-domain-join-success.png)
![Login screen](../screenshots/02-foundation/26-nwfs01-domain-login.png)
![Confirmed in Server Manager](../screenshots/02-foundation/27-nwfs01-domain-joined-confirmed.png)

---

## Step 14 - Create NW-WS-01 VM

Windows 10 Pro, 40 GB disk, 2 GB RAM. Clean install with local account `localadmin` created during OOBE.

![NW-WS-01 created](../screenshots/02-foundation/28-nwws01-vm-created.png)
![Windows 10 installed](../screenshots/02-foundation/29-nwws01-windows10-installed.png)

---

## Step 15 - Configure NW-WS-01

Powered off, switched adapter from NAT to Northwind-NAT. Booted, renamed to NW-WS-01 via Settings > System > About. Set DNS to 10.0.2.10 (NW-DC-01) — IP left on DHCP as workstations do not need a fixed address, but DNS must point to the DC so the domain can be located during join.

No screenshots captured for adapter change and rename.

![DNS configured](../screenshots/02-foundation/32-nwws01-dns-configured.png)

---

## Step 16 - Domain Join NW-WS-01

Joined from an elevated PowerShell prompt on NW-WS-01:

```powershell
Add-Computer -DomainName northwind.local -Credential NORTHWIND\Administrator -Restart
```

Confirmed joined after reboot — `PartOfDomain` returns `True` and the domain reads `northwind.local`:

```powershell
(Get-WmiObject Win32_ComputerSystem) | Select-Object Name, Domain, PartOfDomain
# Domain : northwind.local   PartOfDomain : True
```

> **Lesson learned:** Domain join — and any later domain login — will fail if NW-DC-01 is not running. The workstation relies on the DC for DNS resolution of `northwind.local` and for Kerberos authentication. A login attempt as `NORTHWIND\Administrator` while the DC is off (or before the machine is actually joined) is rejected because the workstation has no way to reach the domain. Always boot the DC first. In production the DC is always on — in a lab you have to start it manually.

![Domain join confirmed](../screenshots/02-foundation/30-nwws01-domain-joined-confirmed.png)

---

## Step 17 - Verify Domain Login from NW-WS-01

Logged in at the workstation as `NORTHWIND\Administrator` (the only domain account that exists at this stage — the 30 employee accounts are created in Phase 3). A successful domain login is itself proof the machine is correctly joined and can authenticate against the DC over Kerberos.

---

## Summary

| Step | Description | Status |
|---|---|---|
| 1 | Create NW-DC-01 VM | Done |
| 2 | Install Windows Server 2016 | Done |
| 3 | Rename to NW-DC-01 | Done |
| 4 | Create NAT Network | Done |
| 5 | Attach NW-DC-01 to NAT Network | Done |
| 6 | Static IP 10.0.2.10 | Done |
| 7 | Install AD DS role | Done |
| 8 | Promote to Domain Controller | Done |
| 9 | Configure DNS | Done |
| 10 | Verify AD and DNS | Done |
| 11 | Create NW-FS-01 VM | Done |
| 12 | Configure NW-FS-01 | Done |
| 13 | Domain join NW-FS-01 | Done |
| 14 | Create NW-WS-01 VM | Done |
| 15 | Configure NW-WS-01 | Done |
| 16 | Domain join NW-WS-01 | Done |
| 17 | Verify domain login | Done |

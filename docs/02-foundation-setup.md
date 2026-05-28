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

**Screenshots:** 01-vm-creation-start.png, 02-vm-hardware-specs.png, 03-vm-storage-settings.png, 04-nwdc01-vm-created.png

---

## Step 2 — Install Windows Server 2016

**Status: Complete**

Installed Windows Server 2016 Standard (Desktop Experience) on NW-DC-01. Desktop Experience selected to get the full GUI — the Server Core option provides command line only and is not suitable for this lab environment.

Installation type: Custom (clean install to the 50 GB virtual disk).
Administrator password set during installation.

**Screenshots:** 05-ws2016-edition-select.png, 06-ws2016-install-type.png, 07-ws2016-first-login.png

---

## Step 3 — Rename Server to NW-DC-01

**Status: Complete**

Server name confirmed as NW-DC-01 post-installation. Verified in System Properties.

**Screenshot:** 09-server-name-confirmed.png

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

**Screenshot:** 10-nat-network-created.png

---

## Step 5 — Attach NW-DC-01 to the NAT Network

**Status: Complete**

Changed NW-DC-01 network adapter from NAT to NAT Network (Northwind-NAT) while the VM was powered off. VM restarted after the change.

**Screenshot:** 11-nwdc01-adapter-nat-network.png

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

**Screenshots:** 12-static-ip-configured.png, 13-ipconfig-verified.png

---

## Step 7 — Install AD DS Role

**Status: Complete**

Installed the Active Directory Domain Services role via Server Manager (Add Roles and Features). No additional features were required beyond those automatically selected by the role installation.

**Screenshot:** 14-adds-role-installed.png

---

## Step 8 — Promote to Domain Controller

**Status: In Progress**

---

## Step 9 — Configure DNS Zone for northwind.local

**Status: Not Started**

---

## Step 10 — Verify AD and DNS

**Status: Not Started**

---

## Step 11 — Create NW-FS-01 VM, Install Server 2016

**Status: Not Started**

---

## Step 12 — Configure NW-FS-01 (Rename, Network, Static IP)

**Status: Not Started**

---

## Step 13 — Domain Join NW-FS-01

**Status: Not Started**

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
| 8 | Promote to Domain Controller | In Progress |
| 9 | Configure DNS zone for northwind.local | Not Started |
| 10 | Verify AD and DNS | Not Started |
| 11 | Create NW-FS-01 VM, install Server 2016 | Not Started |
| 12 | Configure NW-FS-01 | Not Started |
| 13 | Domain join NW-FS-01 | Not Started |
| 14 | Create NW-WS-01 VM, install Windows 10 | Not Started |
| 15 | Configure NW-WS-01 | Not Started |
| 16 | Domain join NW-WS-01 | Not Started |
| 17 | Verify domain login from NW-WS-01 | Not Started |

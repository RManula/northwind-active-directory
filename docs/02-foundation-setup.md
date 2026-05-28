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

All three VMs run on a VirtualBox NAT Network (`10.0.2.0/24`). The NAT Network allows inter-VM communication while still providing internet access through the host.

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

**Screenshots:** `screenshots/02-foundation/01-vm-creation-start.png`, `02-vm-hardware-specs.png`, `03-vm-storage-settings.png`, `04-nwdc01-vm-created.png`

---

## Step 2 — Install Windows Server 2016

**Status: In Progress**

---

## Step 3 — Rename Server to NW-DC-01

**Status: Not Started**

---

## Step 4 — Create the NAT Network in VirtualBox

**Status: Not Started**

---

## Step 5 — Attach NW-DC-01 to the NAT Network

**Status: Not Started**

---

## Step 6 — Assign Static IP 10.0.2.10

**Status: Not Started**

---

## Step 7 — Install AD DS Role

**Status: Not Started**

---

## Step 8 — Promote to Domain Controller

**Status: Not Started**

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
| 2 | Install Windows Server 2016 | In Progress |
| 3 | Rename server to NW-DC-01 | Not Started |
| 4 | Create NAT Network in VirtualBox | Not Started |
| 5 | Attach NW-DC-01 to NAT Network | Not Started |
| 6 | Assign static IP 10.0.2.10 | Not Started |
| 7 | Install AD DS role | Not Started |
| 8 | Promote to Domain Controller | Not Started |
| 9 | Configure DNS zone for northwind.local | Not Started |
| 10 | Verify AD and DNS | Not Started |
| 11 | Create NW-FS-01 VM, install Server 2016 | Not Started |
| 12 | Configure NW-FS-01 | Not Started |
| 13 | Domain join NW-FS-01 | Not Started |
| 14 | Create NW-WS-01 VM, install Windows 10 | Not Started |
| 15 | Configure NW-WS-01 | Not Started |
| 16 | Domain join NW-WS-01 | Not Started |
| 17 | Verify domain login from NW-WS-01 | Not Started |

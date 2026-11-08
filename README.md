#  Graduation Project (Task 2): Domain Controller Integration

A Windows Server 2019 **Domain Controller** (Active Directory and DNS) added to the DMZ of the **FortiGate 7.4.12** multi-zone network, built and tested in **GNS3** and **VMware Workstation**.

The task covers placing the Domain Controller in the DMZ, connecting **PC1** from the Internal zone to it through a dedicated firewall policy, joining PC1 to the domain `YoussufAhmed.local`, and verifying connectivity, name resolution and domain membership.

---

## Table of Contents
1. [Topology](#1-topology)
2. [Requirements](#2-requirements)
3. [Configuration](#3-configuration)
4. [Verification](#4-verification)
5. [Tools](#5-tools)

---

## 1. Topology

![Topology](<Screenshots Task 2/Topology.png>)

| Zone | Interface | Subnet | Hosts |
|---|---|---|---|
| External | port1 | 192.168.1.0/24 | Firewall external IP: 192.168.1.2 |
| DMZ | port2 | 192.168.2.0/24 | **Domain Controller (192.168.2.40)**, plus the Web, DNS, FTP and File servers from Task 1 |
| Internal | port3 | 172.16.10.0/24 | **PC1**, PC2 |

Traffic from PC1 to the Domain Controller follows this path:

`PC1 → IOU2 → FortiGate (port3) → FortiGate (port2) → IOU1 → Domain Controller`

---

## 2. Requirements

| Source | Allowed destination | Services |
|---|---|---|
| **PC1** | Domain Controller | SAMBA, SMB and 7 additional domain services |

Everything else between the two hosts is denied by the implicit deny rule.

---

## 3. Configuration

### 3.1 Domain Controller

| Setting | Value |
|---|---|
| Operating system | Windows Server 2019 |
| Roles | Active Directory Domain Services (AD DS), DNS |
| Domain name | `YoussufAhmed.local` |
| IP address | 192.168.2.40 |
| Zone | DMZ (port2), connected through IOU1 |

PC1 uses the Domain Controller as its DNS server and is joined to the domain `YoussufAhmed.local`.

### 3.2 Firewall policy

| Policy | Direction | Source | Destination | Schedule | Services | Action | NAT |
|---|---|---|---|---|---|---|---|
| PC1-to-Domain Controller | port3 → port2 | PC1 | Domain Controller | always | SAMBA, SMB, +7 | ACCEPT | Disabled |
| implicit_deny | any | all | all | always | ALL | Deny | – |

![Firewall Policy](<Screenshots Task 2/Policies.png>)

---

## 4. Verification

### 4.1 PC1 → Domain Controller (reachability and DNS)

`ping 192.168.2.40` returned 4 replies with 0% packet loss, and `nslookup YoussufAhmed.local` resolved the domain name to 192.168.2.40.

The "DNS request timed out" line at the start of the nslookup output appears because the server address has no reverse (PTR) record. It does not affect name resolution.

![PC1 ping and nslookup](<Screenshots Task 2/PC1 Ping and Request Services from Domain Controller.png>)

### 4.2 PC1 registered in Active Directory

In *Active Directory Users and Computers*, PC1 appears under the **Computers** container of `YoussufAhmed.local`, which confirms it joined the domain.

![PC1 in Active Directory](<Screenshots Task 2/PC1 in Domain Controller Server.png>)

### 4.3 Summary

| Test | Expected | Result |
|---|---|---|
| PC1 → Domain Controller (ping) | Allowed | Pass |
| PC1 → Domain name resolution (nslookup) | Resolves to 192.168.2.40 | Pass |
| PC1 joined to the domain | Computer object in Active Directory | Pass |
| Any other traffic | Denied | Enforced by implicit deny |

---

## 5. Tools

- FortiGate VM 7.4.12 (KVM)
- GNS3 2.2.52
- VMware Workstation
- Windows Server 2019 (DMZ servers)
- Windows 10 (clients)

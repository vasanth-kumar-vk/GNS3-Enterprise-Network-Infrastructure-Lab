# 🏢 GNS3 Enterprise Network Infrastructure Lab

> **Full-stack enterprise network simulation featuring pfSense perimeter firewall, OSPF dynamic routing with MD5 authentication, VLAN segmentation, Windows Server 2022 Active Directory, Ubuntu DMZ web server, and Zabbix 7.0 monitoring — built in GNS3 with real VMware VM integration.**

---

## 📋 Project Overview

This project simulates a production-grade enterprise branch network built entirely from scratch in GNS3 with VMware Workstation. The topology integrates real VMs (pfSense, Windows Server 2022, Ubuntu Server) with GNS3 virtual routers and switches, creating a fully functional lab environment.

The lab covers enterprise networking, system administration, security policy, and infrastructure monitoring — all verified end-to-end with documented test results.

---

## 🗺️ Network Topology

```
Internet (Physical Network)
        |
   pfSenseFW (VMware VM)
        | VMnet2
   HQ Router (C7200) — 10.0.0.0/30
        |
   BR2 Router (C7200) — 10.0.1.0/30
        |
   BR1 Router (C7200) — 10.0.2.0/30
   |                         |
LANSW (IOU L2)          DMZSW (IOU L2)
├── VLAN 10 — HR (PC1, PC2)     |
├── VLAN 20 — IT (PC3, PC4)  Ubuntu Server (VMware VM)
└── VLAN 99 — MGMT             192.168.100.10
       |                       Apache | Zabbix
  DC01 (VMware VM)
  Windows Server 2022
  AD DS | DNS | DHCP
  192.168.99.10
```

---

## 🛠️ Technologies & Stack

| Category | Technology |
|---|---|
| Simulation Platform | GNS3 + GNS3 VM, VMware Workstation Pro 25 |
| Router Platform | Cisco C7200 IOS |
| Switch Platform | Cisco IOU L2 |
| Perimeter Firewall | pfSense CE 2.8.1 |
| Routing Protocol | OSPF Area 0 with MD5 Authentication |
| Switching | 802.1Q Trunking, VLANs 10/20/99/100 |
| Inter-VLAN Routing | Router-on-a-stick (subinterfaces on BR1) |
| Directory Services | Windows Server 2022 — AD DS, DNS, DHCP |
| DMZ Server | Ubuntu Server 24.04 — Apache2 |
| Monitoring | Zabbix 7.0.25 with MySQL backend |
| Remote Management | SSH v2 on all routers |
| SNMP | SNMPv2c on all routers (Zabbix polling) |

---

## 📡 IP Addressing Scheme

| Network | Subnet | Purpose | Gateway |
|---|---|---|---|
| pfSense ↔ HQ | 10.0.0.0/30 | WAN link | — |
| HQ ↔ BR2 | 10.0.1.0/30 | Transit link | — |
| BR2 ↔ BR1 | 10.0.2.0/30 | Transit link | — |
| VLAN 10 (HR) | 192.168.10.0/24 | HR workstations | 192.168.10.1 |
| VLAN 20 (IT) | 192.168.20.0/24 | IT workstations | 192.168.20.1 |
| VLAN 99 (MGMT) | 192.168.99.0/24 | Management / DC01 | 192.168.99.1 |
| VLAN 100 (DMZ) | 192.168.100.0/24 | Ubuntu DMZ server | 192.168.100.1 |

---

## ⚙️ Key Configurations

### pfSense Firewall
- WAN (em0): DHCP from physical network — internet uplink
- LAN (em1): 10.0.0.1/30 — connects to HQ router via VMnet2
- Manual outbound NAT covering 192.168.0.0/16 and 10.0.0.0/8
- Static route: 192.168.0.0/16 via 10.0.0.2 (HQ)
- Firewall rules enforcing DMZ isolation

### OSPF Dynamic Routing
- 3 routers in Area 0: HQ (1.1.1.1), BR2 (2.2.2.2), BR1 (3.3.3.3)
- MD5 authentication on all OSPF interfaces
- `default-information originate always` at HQ propagates default route
- Static default route on HQ: `ip route 0.0.0.0 0.0.0.0 10.0.0.1`

### VLAN Segmentation & Trunking
- VLANs 10, 20, 99 on LANSW — 802.1Q trunk to BR1 fa0/1
- VLAN 100 (DMZ) on DMZSW — access port to BR1 fa2/0
- DHCP relay via `ip helper-address 192.168.99.10` on BR1 subinterfaces

### Active Directory (DC01)
- Domain: lab.local
- OU: IT DEPARTMENT with users and security groups
- DHCP scopes for VLAN 10 and VLAN 20
- PowerShell automation scripts in C:\Scripts\

### Zabbix Monitoring
- 6 hosts monitored: Zabbix server, Ubuntu-Server, DC01, HQ-Router, BR2-Router, BR1-Router
- Cisco routers monitored via SNMPv2c
- Linux/Windows hosts monitored via Zabbix Agent
- Email alerts configured via Gmail SMTP with recovery notifications

---

## 🔒 Security Policy (ACLs on BR1)

| Source VLAN | Destination | Access |
|---|---|---|
| VLAN 10 — HR | VLAN 20 (IT) | ❌ Blocked |
| VLAN 10 — HR | VLAN 100 (DMZ) | ❌ Blocked |
| VLAN 10 — HR | VLAN 99 (MGMT) | ✅ Allowed |
| VLAN 20 — IT | VLAN 10 (HR) | ❌ Blocked |
| VLAN 20 — IT | VLAN 100 (DMZ) | ✅ Allowed |
| VLAN 99 — MGMT | Everywhere | ✅ Full Access |
| VLAN 100 — DMZ | VLAN 10 (HR) | ❌ Blocked |
| VLAN 100 — DMZ | VLAN 99 (MGMT) | ✅ Allowed (responses) |
| VLAN 100 — DMZ | Internet | ✅ Allowed |

---

## ✅ Verification Results

| Test | Source | Destination | Result |
|---|---|---|---|
| OSPF neighbor adjacency | HQ ↔ BR2 ↔ BR1 | All neighbors | ✅ FULL state |
| DHCP lease assignment | PC1–PC4 | DC01 (192.168.99.10) | ✅ Pass |
| Internet access | All VLANs | 8.8.8.8 | ✅ Pass |
| HR blocked from IT | PC1 (VLAN10) | PC3 (VLAN20) | ✅ Blocked |
| HR blocked from DMZ | PC1 (VLAN10) | Ubuntu (DMZ) | ✅ Blocked |
| IT access to DMZ | PC3 (VLAN20) | Ubuntu (DMZ) | ✅ Pass |
| DMZ blocked from HR | Ubuntu | PC1 (192.168.10.x) | ✅ Blocked |
| DMZ access to MGMT | Ubuntu | DC01 (192.168.99.10) | ✅ Pass |
| Apache web server | DC01 browser | http://192.168.100.10 | ✅ Pass |
| SSH to routers | DC01 (PuTTY) | HQ, BR2, BR1 | ✅ Pass |
| Zabbix monitoring | All 6 hosts | Green availability | ✅ Pass |
| Email alerts | Zabbix | Gmail | ✅ Pass |

---

## 📁 Repository Structure

```
gns3-enterprise-network-infrastructure-lab/
├── docs/
│   ├── Enterprise_Branch_Network_Lab_VasanthKumar_v2.docx
│   └── Enterprise_Branch_Network_Lab_VasanthKumar_v2.pdf
└── README.md
```

---

## 🖥️ VMware Network Adapter Setup

| VMnet | Type | Purpose |
|---|---|---|
| VMnet0 | Bridged | pfSense WAN — real internet |
| VMnet2 | Host-only | pfSense LAN ↔ GNS3 HQ cloud |
| VMnet3 | Host-only | DC01 ↔ GNS3 LANSW cloud |
| VMnet4 | Host-only | Ubuntu ↔ GNS3 DMZSW cloud |

---

## 📚 Skills Demonstrated

`OSPF` `MD5 Authentication` `VLANs` `802.1Q Trunking` `Inter-VLAN Routing` `pfSense Firewall` `NAT/PAT` `ACLs` `DMZ Architecture` `Active Directory` `DNS` `DHCP` `DHCP Relay` `SSH Hardening` `SNMP` `Zabbix Monitoring` `Email Alerting` `PowerShell Automation` `Ubuntu Server` `Apache` `GNS3` `VMware`

---

## 👨‍💻 Author

**Vasanth Kumar R**
📧 vasanthkumarvk2855@gmail.com
🔗 [LinkedIn](https://linkedin.com/in/vasanth-kumar-vk55) | [GitHub](https://github.com/vasanth-kumar-vk)
🏅 Cisco Networking Basics Certification — February 2026

---

*This project is part of a hands-on IT infrastructure portfolio built to demonstrate entry-level Network Administrator and System Administrator skills.*

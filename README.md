---
# 🔐 Project 3: Secure Enterprise Network

## 📌 Project Overview

This project implements a **defense-in-depth security architecture** for an enterprise network. Security controls are applied at **Layer 2 (switching)**, **Layer 3 (routing)**, and **management plane** to protect against internal and external threats.

The lab focuses on securing access ports, preventing rogue DHCP servers, mitigating ARP spoofing, restricting inter-VLAN traffic, and enabling secure device management via SSH, SNMPv3, NTP, and Syslog.

---

## 🎯 Objectives

| Security Layer | Technologies Used |
|----------------|-------------------|
| **Layer 2 Security** | Port Security, DHCP Snooping, DAI |
| **Layer 3 Security** | Extended ACLs |
| **Management Security** | SSHv2, SNMPv3, NTP, Syslog |
| **Infrastructure Security** | BPDU Guard, Storm Control |

---

## 🏗️ Network Topology

```
```

![Topology Diagram](./Screenshots/Topology-Network/network-topology-main.png)

---

## 📊 IP Addressing Scheme

| VLAN | Network | Subnet Mask | Virtual Gateway (HSRP) | CORE-SW1 | CORE-SW2 |
|------|---------|-------------|------------------------|----------|----------|
| **VLAN 10 (Mgmt)** | 172.16.0.0/24 | 255.255.255.0 | 172.16.0.1 | 172.16.0.2 | 172.16.0.3 |
| **VLAN 20 (HR)** | 172.16.1.0/24 | 255.255.255.0 | 172.16.1.1 | 172.16.1.2 | 172.16.1.3 |
| **VLAN 30 (Guest)** | 172.16.2.0/24 | 255.255.255.0 | 172.16.2.1 | 172.16.2.2 | 172.16.2.3 |

### HSRP Priority Plan

| Switch | VLAN 10 | VLAN 20 | VLAN 30 |
|--------|:-------:|:-------:|:-------:|
| **CORE-SW1** | Active (150) | Active (150) | Standby (100) |
| **CORE-SW2** | Standby (100) | Standby (100) | Active (150) |

---

## ⚙️ Key Configuration Snippets

### Access Switch — Port Security & DHCP Snooping

```
interface range gi0/0 - 3
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
 no shutdown

ip dhcp snooping
ip dhcp snooping vlan 10,20,30
no ip dhcp snooping information option
```

### Core Switch — HSRP & DHCP

```
interface Vlan10
 ip address 172.16.0.2 255.255.255.0
 standby 10 ip 172.16.0.1
 standby 10 priority 150
 standby 10 preempt
 no shutdown

ip dhcp pool VLAN10
 network 172.16.0.0 255.255.255.0
 default-router 172.16.0.1
 dns-server 8.8.8.8
```

### Core Router — SSH & SNMPv3

```
ip domain-name secure.local
crypto key generate rsa modulus 2048
ip ssh version 2
username admin secret Cisco123!
line vty 0 4
 transport input ssh
 login local

snmp-server group SECURE v3 priv
snmp-server user admin SECURE v3 auth sha Cisco123! priv aes 128 Cisco123!
```

**[📄 Full Configs](./configs/)**

---

## ✅ Verification Commands

| Device | Command | Purpose |
|--------|---------|---------|
| **Access Switch** | `show port-security` | Port security status |
| | `show ip dhcp snooping` | DHCP snooping status |
| | `show interface trunk` | Trunk verification |
| **Core Switch** | `show standby brief` | HSRP Active/Standby |
| | `show ip dhcp binding` | DHCP leases |
| | `show interface trunk` | Trunk status |
| **Core Router** | `show ip ssh` | SSH status |
| | `show ntp status` | NTP sync |
| | `show logging` | Syslog status |
| **End Host** | `ping 172.16.0.1` | Gateway reachability |

---

## 📸 Screenshots

| Figure | Description | Command |
|--------|-------------|---------|
| 1 | Port Security & Sticky MACs | `show port-security` |
| 2 | DHCP Snooping Status | `show ip dhcp snooping` |
| 3 | HSRP Active/Standby | `show standby brief` |
| 4 | DHCP Bindings | `show ip dhcp binding` |
| 5 | SSH v2 Status | `show ip ssh` |
| 6 | NTP Synchronization | `show ntp status` |
| 7 | Syslog Logging | `show logging` |
| 8 | Trunk Interfaces | `show interface trunk` |
| 9 | End-to-End Ping Test | `ping 172.16.0.1` |

---

## 📚 Lessons Learned

| Issue | Solution |
|-------|----------|
| BPDU Guard on trunk ports causing err-disable | Remove BPDU Guard on uplink trunks |
| DAI blocking ARP causing ping failures | Disable DAI or trust uplink interfaces |
| DHCP Option 82 drops in GNS3 | `no ip dhcp snooping information option` |
| Duplicate IP conflict on SVIs | Unique physical IPs, common Virtual IP |
| HSRP not forming | Preempt with correct priorities |

---

## 🚀 Future Improvements

- Implement AAA (TACACS+) for centralized authentication
- Deploy IPS/IDS for threat detection
- Implement 802.1X for port-based authentication
- Add Firewall integration

---

## 📂 Repository Structure

```
Project3-Secure-Enterprise/
├── README.md
├── configs/
│   ├── ACC-SW1.txt
│   ├── ACC-SW2.txt
│   ├── CORE-SW1.txt
│   ├── CORE-SW2.txt
│   └── CORE-Router-1.txt
├── Screenshots/
│   ├── ACCESS-SW1/
│   ├── ACCESS-SW2/
│   ├── Core-SW1/
│   ├── Core-SW2/
│   ├── Core-Router1/
│   └── Topology-Network/
└── topology/
    └── network-diagram.png
```

## 📄 License

MIT License — see [LICENSE](./LICENSE) file for details.

---

⭐ **If you find this useful, please consider giving it a star!** ⭐
---

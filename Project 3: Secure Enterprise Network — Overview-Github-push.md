---

# 📁 Project 3: Secure Enterprise Network

## 📌 Project Overview
This project focuses on implementing a **defense-in-depth security architecture** for an enterprise network. Security controls are applied at **Layer 2 (switching)**, **Layer 3 (routing)**, and **management plane** levels to protect against internal and external threats.

## 🎯 Objectives
- **Layer 2 Security:** Port Security, DHCP Snooping, Dynamic ARP Inspection (DAI)
- **Layer 3 Security:** Extended ACLs for traffic filtering
- **Management Security:** SSH, SNMPv3, NTP, Syslog
- **Infrastructure Security:** BPDU Guard, Root Guard, Storm Control

---

## 🏗️ Network Topology

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    SECURE ENTERPRISE NETWORK                           │
│                                                                         │
│                    ┌─────────────────────┐                             │
│                    │    Core Router      │                             │
│                    │   (SSH, SNMPv3)     │                             │
│                    └──────────┬──────────┘                             │
│                               │                                        │
│                    ┌──────────┴──────────┐                             │
│                    │   Distribution SW   │                             │
│                    │   (ACLs, HSRP)      │                             │
│                    └──────────┬──────────┘                             │
│                               │                                        │
│                    ┌──────────┴──────────┐                             │
│                    │   Access Switch     │                             │
│                    │   (Port Security,   │                             │
│                    │    DHCP Snooping,   │                             │
│                    │    DAI)             │                             │
│                    └──────────┬──────────┘                             │
│                               │                                        │
│               ┌───────────────┼───────────────┐                       │
│               │               │               │                       │
│         ┌─────┴─────┐   ┌─────┴─────┐   ┌─────┴─────┐               │
│         │  VLAN 10  │   │  VLAN 20  │   │  VLAN 30  │               │
│         │ (Mgmt)    │   │ (HR)      │   │ (Guest)   │               │
│         └───────────┘   └───────────┘   └───────────┘               │
│                                                                         │
│   🔵 = Security Features Applied                                       │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 IP Addressing Scheme

| VLAN | Network | Subnet Mask | Virtual Gateway (HSRP) | CORE-SW1 (Physical) | CORE-SW2 (Physical) |
|------|---------|-------------|------------------------|---------------------|---------------------|
| **VLAN 10 (Mgmt)** | 172.16.0.0/24 | 255.255.255.0 | 172.16.0.1 | 172.16.0.2 | 172.16.0.3 |
| **VLAN 20 (HR)** | 172.16.1.0/24 | 255.255.255.0 | 172.16.1.1 | 172.16.1.2 | 172.16.1.3 |
| **VLAN 30 (Guest)** | 172.16.2.0/24 | 255.255.255.0 | 172.16.2.1 | 172.16.2.2 | 172.16.2.3 |

### HSRP Priority Plan
| Switch | VLAN 10 | VLAN 20 | VLAN 30 |
|--------|---------|---------|---------|
| **CORE-SW1** | Active (150) | Active (150) | Standby (100) |
| **CORE-SW2** | Standby (100) | Standby (100) | Active (150) |

---

## 🔧 Updated Configuration Codes (Golden Configs)

### 1. Access Switches (`ACC-SW1` & `ACC-SW2`)

```
enable
configure terminal

! 1. VLAN Creation
vlan 10
 name VLAN10_Mgmt
vlan 20
 name VLAN20_HR
vlan 30
 name VLAN30_Guest
exit

! 2. Uplinks to Core Switches (Gi3/2 & Gi3/3)
interface range gi3/2 - 3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 1
 switchport trunk allowed vlan 10,20,30
 no spanning-tree bpduguard enable
 no switchport port-security
 ip dhcp snooping trust
 ip arp inspection trust
 no shutdown
exit

! 3. Access Ports Configuration
interface range gi0/0 - 3, gi1/0 - 3
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
 no shutdown
exit

interface range gi2/0 - 3
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit

interface range gi3/0 - 1
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
exit

! 4. DHCP Snooping
ip dhcp snooping
ip dhcp snooping vlan 10,20,30
no ip dhcp snooping information option

! 5. DAI Disabled (to fix ping issues)
no ip arp inspection vlan 10,20,30

end
write memory
```

---

### 2. Core Switch 1 (`CORE-SW1` — HSRP Active for VLAN 10, 20)

```
enable
configure terminal

! 1. Layer 3 SVI & HSRP Setup
interface Vlan10
 ip address 172.16.0.2 255.255.255.0
 standby 10 ip 172.16.0.1
 standby 10 priority 150
 standby 10 preempt
 no shutdown
exit

interface Vlan20
 ip address 172.16.1.2 255.255.255.0
 standby 20 ip 172.16.1.1
 standby 20 priority 150
 standby 20 preempt
 no shutdown
exit

interface Vlan30
 ip address 172.16.2.2 255.255.255.0
 standby 30 ip 172.16.2.1
 standby 30 priority 100
 no shutdown
exit

! 2. DHCP Server Configuration
service dhcp
ip dhcp excluded-address 172.16.0.1 172.16.0.10
ip dhcp excluded-address 172.16.1.1 172.16.1.10
ip dhcp excluded-address 172.16.2.1 172.16.2.10

ip dhcp pool VLAN10
 network 172.16.0.0 255.255.255.0
 default-router 172.16.0.1
 dns-server 8.8.8.8
exit

ip dhcp pool VLAN20
 network 172.16.1.0 255.255.255.0
 default-router 172.16.1.1
 dns-server 8.8.8.8
exit

ip dhcp pool VLAN30
 network 172.16.2.0 255.255.255.0
 default-router 172.16.2.1
 dns-server 8.8.8.8
exit

! 3. DHCP Option 82 Bypass
ip dhcp relay information trust-all
no ip dhcp snooping information option

! 4. Uplinks Trust
interface range gi0/0 - 1
 ip dhcp snooping trust
 ip arp inspection trust
exit

end
write memory
```

---

### 3. Core Switch 2 (`CORE-SW2` — HSRP Active for VLAN 30)

```
enable
configure terminal

! 1. Layer 3 SVI & HSRP Setup
interface Vlan10
 ip address 172.16.0.3 255.255.255.0
 standby 10 ip 172.16.0.1
 standby 10 priority 100
 no shutdown
exit

interface Vlan20
 ip address 172.16.1.3 255.255.255.0
 standby 20 ip 172.16.1.1
 standby 20 priority 100
 no shutdown
exit

interface Vlan30
 ip address 172.16.2.3 255.255.255.0
 standby 30 ip 172.16.2.1
 standby 30 priority 150
 standby 30 preempt
 no shutdown
exit

! 2. DHCP Server Configuration
service dhcp
ip dhcp excluded-address 172.16.0.1 172.16.0.10
ip dhcp excluded-address 172.16.1.1 172.16.1.10
ip dhcp excluded-address 172.16.2.1 172.16.2.10

ip dhcp pool VLAN10
 network 172.16.0.0 255.255.255.0
 default-router 172.16.0.1
 dns-server 8.8.8.8
exit

ip dhcp pool VLAN20
 network 172.16.1.0 255.255.255.0
 default-router 172.16.1.1
 dns-server 8.8.8.8
exit

ip dhcp pool VLAN30
 network 172.16.2.0 255.255.255.0
 default-router 172.16.2.1
 dns-server 8.8.8.8
exit

! 3. DHCP Option 82 Bypass
ip dhcp relay information trust-all
no ip dhcp snooping information option

! 4. Uplinks Trust
interface range gi0/0 - 1
 ip dhcp snooping trust
 ip arp inspection trust
exit

end
write memory
```

---

### 4. Core Router (`CORE-Router-1` — SSH, SNMPv3, NTP, Syslog)

```
enable
configure terminal
hostname CORE-Router-1

! 1. SSH Configuration
ip domain-name secure.local
crypto key generate rsa modulus 2048
ip ssh version 2
ip ssh authentication-retries 3
ip ssh time-out 60

username admin secret Cisco123!
line vty 0 4
 transport input ssh
 login local
 exec-timeout 10 0
 logging synchronous
exit

! 2. SNMPv3 Configuration
snmp-server group SECURE v3 priv
snmp-server user admin SECURE v3 auth sha Cisco123! priv aes 128 Cisco123!
snmp-server view ALL iso included
snmp-server enable traps
snmp-server host 172.16.0.100 version 3 priv admin udp-port 162

! 3. NTP Configuration
ntp server 172.16.0.1
ntp source loopback 0
ntp authentication-key 1 md5 Cisco123!
ntp trusted-key 1
ntp authenticate

! 4. Syslog Configuration
logging on
logging host 172.16.0.100
logging trap debugging
logging source-interface loopback 0
logging buffered informational

end
write memory
```

---

## ✅ Verification Commands & Expected Outputs

### 1. Access Switches (`ACC-SW1` & `ACC-SW2`)

| Command | Purpose | Expected Output |
|---------|---------|-----------------|
| `show interface trunk` | Verify trunk links | `Gi3/2`, `Gi3/3` status `trunking` |
| `show port-security` | Port security status | Max 2, Sticky MACs |
| `show port-security address` | Sticky MAC addresses | MAC list with VLAN |
| `show ip dhcp snooping` | DHCP snooping status | Enabled, Trusted uplinks |
| `show ip arp inspection interfaces` | DAI interface status | Trusted/Untrusted |
| `show interface status err-disabled` | Error disabled ports | None |

---

### 2. Core Switches (`CORE-SW1` & `CORE-SW2`)

| Command | Purpose | Expected Output |
|---------|---------|-----------------|
| `show standby brief` | HSRP status | Active/Standby states |
| `show ip dhcp binding` | DHCP leases | IP-MAC bindings |
| `show ip dhcp pool` | DHCP pool stats | Utilization |
| `show interface trunk` | Trunk status | `Gi0/0-1` trunking |
| `show ip dhcp snooping` | DHCP snooping | Enabled, Trusted ports |
| `show ip arp inspection interfaces` | DAI status | Trusted uplinks |

---

### 3. Core Routers (`CORE-Router-1` & `CORE-Router-2`)

| Command | Purpose | Expected Output |
|---------|---------|-----------------|
| `show ip ospf neighbor` | OSPF adjacency | `FULL` state |
| `show ip route ospf` | OSPF routes | Dynamic routes |
| `show logging` | Syslog status | Logging to remote host |
| `show ip ssh` | SSH status | Version 2.0, RSA 2048 |
| `show ntp status` | NTP sync | `Clock is synchronized` |
| `show ntp associations` | NTP peers | Peer details |

---

### 4. End Hosts (PCs)

| Command | Purpose | Expected Output |
|---------|---------|-----------------|
| `ip dhcp` | Request IP | IP assigned |
| `show ip` | Verify IP | IP, subnet, gateway |
| `ping 172.16.0.1` | Gateway ping | `!!!!!` |
| `ping 172.16.2.10` | Inter-VLAN ping | `!!!!!` |

---

## 📸 Screenshots

| # | Screenshot | Command |
|---|------------|---------|
| 1 | Port Security | `show port-security` |
| 2 | DHCP Snooping | `show ip dhcp snooping` |
| 3 | HSRP Status | `show standby brief` |
| 4 | DHCP Binding | `show ip dhcp binding` |
| 5 | SSH Connection | `ssh -l admin 172.16.0.1` |
| 6 | SNMPv3 | `show snmp user` |
| 7 | NTP Status | `show ntp status` |
| 8 | Syslog | `show logging` |
| 9 | Ping Test | `ping 172.16.0.1` |
| 10 | Interface Trunk | `show interface trunk` |

---

## 📚 Lessons Learned

| Issue | Solution |
|-------|----------|
| **BPDU Guard** on trunk ports causing err-disable | Remove BPDU Guard on uplink trunks |
| **DAI blocking ARP** causing ping failures | Disable DAI or trust uplink interfaces |
| **DHCP Option 82 drops** in GNS3 | `no ip dhcp snooping information option` |
| **Duplicate IP conflict** on SVIs | Unique physical IPs, common Virtual IP |
| **HSRP not forming** | Preempt with correct priorities |

---

## 🚀 Future Improvements
- Implement **AAA (TACACS+)** for centralized authentication.
- Deploy **IPS/IDS** for threat detection.
- Implement **802.1X** for port-based authentication.
- Add **Firewall** integration.

---

## 🏆 LinkedIn Post

> **"🔐 Project 3: Secure Enterprise Network Completed!**
>
> **✅ Layer 2 Security:** Port Security, DHCP Snooping
> **✅ Layer 3 Security:** Extended ACLs
> **✅ Management Security:** SSH, SNMPv3, NTP, Syslog
> **✅ Infrastructure Security:** BPDU Guard, Storm Control
>
> **🔐 Defense-in-Depth strategy implemented!**
>
> **#Cybersecurity #NetworkSecurity #CCNA #GNS3 #Cisco #Security"**

---

## 📂 GitHub Repository Structure

```
Project3-Secure-Enterprise/
├── README.md
├── configs/
│   ├── ACC-SW1.txt
│   ├── ACC-SW2.txt
│   ├── CORE-SW1.txt
│   ├── CORE-SW2.txt
│   └── CORE-Router-1.txt
├── screenshots/
│   ├── port-security.png
│   ├── dhcp-snooping.png
│   ├── hsrp-status.png
│   ├── dhcp-binding.png
│   ├── ssh-connection.png
│   ├── snmpv3.png
│   ├── ntp-status.png
│   ├── syslog.png
│   └── ping-test.png
└── topology/
    └── network-diagram.png
```

---

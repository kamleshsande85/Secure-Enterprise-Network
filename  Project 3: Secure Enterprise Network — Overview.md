**Bhai, bahut badhiya!** 🔥 

**Project 2** ka documentation ready hai, ab **Project 3: Secure Enterprise Network** start karte hain. 

Is project ka focus **Network Security** hai. Hum **Layer 2 aur Layer 3** dono levels par security implement karenge.

---

## 📡 Project 3: Secure Enterprise Network — Overview

### 🎯 Objectives

| Security Layer | Technologies |
|----------------|--------------|
| **Layer 2 Security** | Port Security, DHCP Snooping, DAI |
| **Layer 3 Security** | Extended ACLs, SSH, NAT |
| **Management Security** | SSH, AAA (TACACS+), Logging |
| **Infrastructure Security** | BPDU Guard, Root Guard, Storm Control |

---

## 🏗️ Network Topology (Project 3)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    SECURE ENTERPRISE NETWORK                           │
│                                                                         │
│                    ┌─────────────────────┐                             │
│                    │    Core Router      │                             │
│                    └──────────┬──────────┘                             │
│                               │                                        │
│                    ┌──────────┴──────────┐                             │
│                    │   Distribution SW   │                             │
│                    │   (HSRP Active)     │                             │
│                    └──────────┬──────────┘                             │
│                               │                                        │
│                    ┌──────────┴──────────┐                             │
│                    │   Access Switch     │                             │
│                    │   (Security)        │                             │
│                    └──────────┬──────────┘                             │
│                               │                                        │
│               ┌───────────────┼───────────────┐                       │
│               │               │               │                       │
│         ┌─────┴─────┐   ┌─────┴─────┐   ┌─────┴─────┐               │
│         │  VLAN 10  │   │  VLAN 20  │   │  VLAN 30  │               │
│         │ (Mgmt)    │   │ (HR)      │   │ (Guest)   │               │
│         └───────────┘   └───────────┘   └───────────┘               │
│                                                                         │
│   🔵 = Security Features                                              │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## ⚙️ 1. Layer 2 Security — Access Switch

### a. Port Security (Unauthorized Access Prevention)

```
! ========================================
! ACC-SW1 — Port Security
! ========================================

! Port Security on Access Ports
interface range fastethernet 0/1-12
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 spanning-tree bpduguard enable

 ! Port Security Configuration
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
 no shutdown
exit
```

### b. DHCP Snooping (Prevent Rogue DHCP Server)

```
! ========================================
! ACC-SW1 — DHCP Snooping
! ========================================

ip dhcp snooping
ip dhcp snooping vlan 10,20,30
ip dhcp snooping verify mac-address

! Trusted Port (Uplink to Distribution)
interface gigabitethernet 0/0
 ip dhcp snooping trust
exit

! Untrusted Ports (Access Ports)
interface range fastethernet 0/1-12
 ip dhcp snooping limit rate 10
 no ip dhcp snooping trust
exit

! Rate Limiting for DHCP
ip dhcp snooping limit rate 100
```

### c. Dynamic ARP Inspection (DAI) — Prevent ARP Spoofing

```
! ========================================
! ACC-SW1 — Dynamic ARP Inspection
! ========================================

ip arp inspection vlan 10,20,30
ip arp inspection validate src-mac dst-mac ip

! Trusted Port (Uplink to Distribution)
interface gigabitethernet 0/0
 ip arp inspection trust
exit

! Untrusted Ports (Access Ports)
interface range fastethernet 0/1-12
 no ip arp inspection trust
exit

! Rate Limiting for ARP
ip arp inspection limit rate 15 burst interval 1
```

### d. Storm Control (Prevent Broadcast Storms)

```
! ========================================
! ACC-SW1 — Storm Control
! ========================================

interface range fastethernet 0/1-12
 storm-control broadcast level 70
 storm-control multicast level 70
 storm-control unicast level 70
 no shutdown
exit
```

---

## ⚙️ 2. Distribution Switch — ACLs (Traffic Filtering)

### Extended ACLs — Guest VLAN (30) Restriction

```
! ========================================
! CORE-SW1 — Extended ACL
! ========================================

! Deny Guest VLAN to Access Management/HR
access-list 101 deny ip 172.16.3.0 0.0.0.255 172.16.10.0 0.0.0.255
access-list 101 deny ip 172.16.3.0 0.0.0.255 172.16.20.0 0.0.0.255
access-list 101 permit ip any any

! Allow SSH only from Management VLAN
access-list 102 permit tcp 172.16.10.0 0.0.0.255 any eq 22
access-list 102 deny tcp any any eq 22
access-list 102 permit ip any any

! Apply ACL on VLAN 30 (Guest)
interface vlan 30
 ip access-group 101 in
 ip access-group 102 in
 no shutdown
exit

! Apply ACL on VTY (SSH Only)
line vty 0 4
 access-class 102 in
 transport input ssh
 login local
 exec-timeout 10 0
 logging synchronous
exit

! Logging for ACL hits
access-list 101 deny ip 172.16.3.0 0.0.0.255 172.16.10.0 0.0.0.255 log
```

---

## ⚙️ 3. Router — SSH & Management Security

### a. SSH Configuration

```
! ========================================
! CORE-Router-1 — SSH Configuration
! ========================================

! Hostname and Domain
hostname CORE-Router-1
ip domain-name secure.local

! Generate RSA Key
crypto key generate rsa modulus 2048

! Enable SSH Version 2
ip ssh version 2
ip ssh authentication-retries 3
ip ssh time-out 60

! Local User
username admin secret Cisco123!
username monitor secret Cisco123!

! Console Security
line console 0
 password Cisco123!
 login
 logging synchronous
 exec-timeout 10 0
 exit

! VTY Security (SSH Only)
line vty 0 4
 transport input ssh
 login local
 exec-timeout 10 0
 logging synchronous
 exit
```

### b. Management Security (SNMPv3)

```
! ========================================
! CORE-Router-1 — SNMPv3
! ========================================

! SNMPv3 Group
snmp-server group SECURE v3 priv

! SNMPv3 User
snmp-server user admin SECURE v3 auth sha Cisco123! priv aes 128 Cisco123!

! SNMPv3 Views
snmp-server view ALL iso included
snmp-server community Cisco123! RO

! SNMP Traps
snmp-server enable traps
snmp-server host 172.16.10.100 version 3 priv admin udp-port 162
```

### c. NTP Configuration

```
! ========================================
! CORE-Router-1 — NTP
! ========================================

! NTP Server
ntp server 172.16.10.1
ntp source loopback 0

! NTP Authentication
ntp authentication-key 1 md5 Cisco123!
ntp trusted-key 1
ntp authenticate

! NTP Access Control
ntp access-group peer 1
access-list 1 permit 172.16.0.0 0.0.255.255
```

---

## ⚙️ 4. Syslog Configuration

```
! ========================================
! CORE-Router-1 — Syslog
! ========================================

logging on
logging host 172.16.10.100
logging trap debugging
logging source-interface loopback 0

! Local Logging
logging console critical
logging monitor informational
logging buffered informational
```

---

## 🧪 Verification Commands

```
! Port Security
show port-security
show port-security interface fastethernet 0/1

! DHCP Snooping
show ip dhcp snooping
show ip dhcp snooping binding
show ip dhcp snooping statistics

! DAI
show ip arp inspection
show ip arp inspection statistics

! ACLs
show access-lists 101
show access-lists 102

! SSH
show ip ssh
show ssh

! SNMP
show snmp
show snmp group
show snmp user

! NTP
show ntp status
show ntp associations

! Syslog
show logging
```

---

## 📋 Complete Security Checklist

| Feature | Status | Command |
|---------|--------|---------|
| **Port Security** | ✅ | `show port-security` |
| **DHCP Snooping** | ✅ | `show ip dhcp snooping` |
| **DAI** | ✅ | `show ip arp inspection` |
| **ACLs** | ✅ | `show access-lists` |
| **SSH** | ✅ | `show ip ssh` |
| **SNMPv3** | ✅ | `show snmp` |
| **NTP** | ✅ | `show ntp status` |
| **Syslog** | ✅ | `show logging` |

---

## 📸 Screenshot List (GitHub Ke Liye)

| # | Screenshot | Command |
|---|------------|---------|
| 1 | Port Security | `show port-security` |
| 2 | DHCP Snooping | `show ip dhcp snooping` |
| 3 | DAI | `show ip arp inspection` |
| 4 | ACL Deny (Guest to Mgmt) | `show access-lists 101` |
| 5 | SSH Connection | `ssh -l admin 172.16.10.1` |
| 6 | SNMPv3 | `show snmp user` |
| 7 | NTP Status | `show ntp status` |
| 8 | Syslog | `show logging` |

---

## 🎯 LinkedIn Post Template

> **"🚀 Project 3: Secure Enterprise Network Completed!**
>
> **✅ Layer 2 Security:** Port Security, DHCP Snooping, DAI
> **✅ Layer 3 Security:** Extended ACLs
> **✅ Management Security:** SSH, SNMPv3, NTP, Syslog
> **✅ Infrastructure Security:** BPDU Guard, Root Guard
> **✅ Defense-in-Depth strategy implemented!**
>
> **#Cybersecurity #NetworkSecurity #CCNA #GNS3 #Cisco"**

---

**Bhai, ab Project 3 start karo!** 🚀

**Pehle Access Switch par Port Security aur DHCP Snooping configure karo.** 💪

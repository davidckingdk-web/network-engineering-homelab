# Completed – 03 DHCP & Inter-VLAN Routing

## 1. Overview

This document confirms the successful implementation of DHCP on R1 and inter-VLAN routing between:

- VLAN 10 (Users)
- VLAN 99 (Management)

The Linux host received a DHCP address and demonstrated full connectivity across VLANs.

---

## 2. Final Device Configurations

### 2.1 R1 – Relevant DHCP & Interface Configuration

```
interface FastEthernet0/0
 ip nat inside
 ip virtual-reassembly in

interface FastEthernet0/0.1
 encapsulation dot1Q 1 native
 ip address 10.0.0.2 255.255.255.252

interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface FastEthernet0/0.99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0

ip dhcp excluded-address 192.168.10.1 192.168.10.9
ip dhcp pool HostIPs
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
```

### 2.2 SW1 – Relevant VLAN & Interface Configuration

```
interface FastEthernet0/1
 description Trunk to R1
 switchport trunk allowed vlan 1,10,99
 switchport mode trunk

interface FastEthernet0/10
 description User WorkStation
 switchport access vlan 10
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 spanning-tree portfast

interface Vlan99
 ip address 192.168.99.2 255.255.255.0
```

---

## 3. Verification

### 3.1 DHCP Binding on R1

```
R1#show ip dhcp binding
IP address       Client-ID                Lease expiration        Type
192.168.10.10    0184.2afd.3e9d.15        Nov 27 2025 02:15 PM    Automatic
```

### 3.2 Connectivity Tests from R1

```
R1#ping 192.168.10.1
!!!!!  (100% success)

R1#ping 192.168.99.1
!!!!!  (100% success)

R1#ping 192.168.99.2
..!!!  (initial ARP resolution)
!!!!!  (100% success on second ping)
```

### 3.3 SW1 – VLAN / Interface State

#### 3.3.1 `show ip interface brief`
```
Vlan99     192.168.99.2    up/up
Fa0/1      trunk link      up/up
Fa0/10     access port     up/up
```

#### 3.3.2 `show vlan`
```
10  Users        active  Fa0/10
99  Management   active
999 Parking      active (all unused ports)
```

#### 3.3.3 `show interfaces switchport Fa0/10`
```
Access Mode VLAN: 10 (Users)
Operational Mode: static access
Portfast: enabled
Sticky MAC: enabled
```

---

## 4. Linux Host Verification

### 4.1 IP Address from DHCP

```
inet 192.168.10.10/24 dev enp0s31f6
```

### 4.2 Routing Table

```
default via 192.168.10.1 dev enp0s31f6
192.168.10.0/24 dev enp0s31f6 proto kernel
```

### 4.3 ARP Table

```
192.168.10.1 lladdr 44:e4:d9:90:c0:2e REACHABLE
```

### 4.4 Ping Tests

```
ping -c 4 192.168.10.1  → 100% success
ping -c 4 192.168.99.1  → 100% success
```

---

## 5. Results Summary

- DHCP is working and leases are issued correctly.  
- R1 performs inter-VLAN routing successfully.  
- SW1 VLANs, trunk, and access ports are correctly configured.  
- The Linux host receives the correct IP from DHCP and reaches all devices.

---

## 6. Notes & Lessons Learned

- First ping failures to SW1 were due to ARP resolution, which is expected behavior.
- Ensuring trunk allowed VLANs match router subinterfaces is critical.
- Testing both router-level and end-host connectivity gives full validation of routing.
- DHCP verification requires checking bindings **and** confirming the host’s routing table.
- VLAN 10 + VLAN 99 functional end‑to‑end connectivity proves the inter-VLAN routing design is correct.

```

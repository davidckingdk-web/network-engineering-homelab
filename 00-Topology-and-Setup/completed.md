# Completed – Ticket 00: Lab Topology & Setup

## Overview
This document contains the completed work for Ticket 00.  
It includes the finalized physical and logical topology, IP addressing, cabling notes, and verification outputs.

---

## 1. Physical Topology Diagram
**File:** [topology.png](topology.png)

---

## 2. Logical VLAN & IP Topology

### VLANs Used
| VLAN | Name          | Purpose              |
|------|---------------|----------------------|
| 1    | Native        | Trunk native VLAN    |
| 10   | Users         | Host network         |
| 99   | Management    | Switch & router mgmt |
| 999  | Parking       | Disabled ports       |

---

## 3. IP Addressing Summary

### **Router R1**
| Interface        | VLAN | IP Address        | Notes        |
|------------------|------|-------------------|--------------|
| Fa0/0.1          | 1    | 10.0.0.2/30       | To ISP       |
| Fa0/0.10         | 10   | 192.168.10.1/24   | DHCP subnet  |
| Fa0/0.99         | 99   | 192.168.99.1/24   | Mgmt VLAN    |

---

### **Switch SW1**
| Interface | Mode   | VLAN | Notes                        |
|-----------|--------|------|------------------------------|
| Fa0/1     | Trunk  | 1,10,99 | To R1 router             |
| Fa0/10    | Access | 10   | Host Ubuntu                 |
| Fa0/2-48  | Access | 999  | Shutdown parking VLAN       |

---

### **Host (Ubuntu)**
| Interface | IP Mode | Address        |
|-----------|---------|----------------|
| ensX      | DHCP    | 192.168.10.10   |

---

## 4. Cabling & Interfaces Used

- **Ubuntu Host** → SW1 Fa0/10  
- **SW1 Fa0/1** → R1 Fa0/0  
- **R1 Fa0/1** → ISP modem (future WAN)  
- All other switch ports shutdown & assigned to VLAN 999  

---

## 5. Verification Outputs

### R1 – `show ip interface brief`

```text
R1#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES NVRAM  up                    up
FastEthernet0/0.1          10.0.0.2        YES NVRAM  up                    up
FastEthernet0/0.10         192.168.10.1    YES NVRAM  up                    up
FastEthernet0/0.99         192.168.99.1    YES NVRAM  up                    up
FastEthernet0/1            unassigned      YES NVRAM  administratively down down
NVI0                       unassigned      YES unset  administratively down down
```

---

### SW1 – `show ip interface brief`

```text
SW1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
Vlan1                  unassigned      YES NVRAM  up                    up
Vlan99                 192.168.99.2    YES NVRAM  up                    up
FastEthernet0/1        unassigned      YES unset  up                    up
FastEthernet0/2        unassigned      YES unset  administratively down down
FastEthernet0/3        unassigned      YES unset  administratively down down
FastEthernet0/4        unassigned      YES unset  administratively down down
FastEthernet0/5        unassigned      YES unset  administratively down down
FastEthernet0/6        unassigned      YES unset  administratively down down
FastEthernet0/7        unassigned      YES unset  administratively down down
FastEthernet0/8        unassigned      YES unset  administratively down down
FastEthernet0/9        unassigned      YES unset  administratively down down
FastEthernet0/10       unassigned      YES unset  up                    up
FastEthernet0/11       unassigned      YES unset  administratively down down
FastEthernet0/12       unassigned      YES unset  administratively down down
FastEthernet0/13       unassigned      YES unset  administratively down down
FastEthernet0/14       unassigned      YES unset  administratively down down
FastEthernet0/15       unassigned      YES unset  administratively down down
FastEthernet0/16       unassigned      YES unset  administratively down down
FastEthernet0/17       unassigned      YES unset  administratively down down
FastEthernet0/18       unassigned      YES unset  administratively down down
FastEthernet0/19       unassigned      YES unset  administratively down down
FastEthernet0/20       unassigned      YES unset  administratively down down
FastEthernet0/21       unassigned      YES unset  administratively down down
FastEthernet0/22       unassigned      YES unset  administratively down down
FastEthernet0/23       unassigned      YES unset  administratively down down
FastEthernet0/24       unassigned      YES unset  administratively down down
FastEthernet0/25       unassigned      YES unset  administratively down down
FastEthernet0/26       unassigned      YES unset  administratively down down
FastEthernet0/27       unassigned      YES unset  administratively down down
FastEthernet0/28       unassigned      YES unset  administratively down down
FastEthernet0/29       unassigned      YES unset  administratively down down
FastEthernet0/30       unassigned      YES unset  administratively down down
FastEthernet0/31       unassigned      YES unset  administratively down down
FastEthernet0/32       unassigned      YES unset  administratively down down
FastEthernet0/33       unassigned      YES unset  administratively down down
FastEthernet0/34       unassigned      YES unset  administratively down down
FastEthernet0/35       unassigned      YES unset  administratively down down
FastEthernet0/36       unassigned      YES unset  administratively down down
FastEthernet0/37       unassigned      YES unset  administratively down down
FastEthernet0/38       unassigned      YES unset  administratively down down
FastEthernet0/39       unassigned      YES unset  administratively down down
FastEthernet0/40       unassigned      YES unset  administratively down down
FastEthernet0/41       unassigned      YES unset  administratively down down
FastEthernet0/42       unassigned      YES unset  administratively down down
FastEthernet0/43       unassigned      YES unset  administratively down down
FastEthernet0/44       unassigned      YES unset  administratively down down
FastEthernet0/45       unassigned      YES unset  administratively down down
FastEthernet0/46       unassigned      YES unset  administratively down down
FastEthernet0/47       unassigned      YES unset  administratively down down
FastEthernet0/48       unassigned      YES unset  administratively down down
GigabitEthernet0/1     unassigned      YES unset  administratively down down
GigabitEthernet0/2     unassigned      YES unset  administratively down down
GigabitEthernet0/3     unassigned      YES unset  down                  down
GigabitEthernet0/4     unassigned      YES unset  administratively down down
```

---

### SW1 – `show vlan brief`

```text
SW1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/3
10   Users                            active    Fa0/10
99   Management                       active
999  Parking                          active    Fa0/2, Fa0/3, Fa0/4, Fa0/5
                                              Fa0/6, Fa0/7, Fa0/8, Fa0/9
                                              Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                              Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                              Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                              Fa0/23, Fa0/24, Fa0/25, Fa0/26
                                              Fa0/27, Fa0/28, Fa0/29, Fa0/30
                                              Fa0/31, Fa0/32, Fa0/33, Fa0/34
                                              Fa0/35, Fa0/36, Fa0/37, Fa0/38
                                              Fa0/39, Fa0/40, Fa0/41, Fa0/42
                                              Fa0/43, Fa0/44, Fa0/45, Fa0/46
                                              Fa0/47, Fa0/48, Gi0/1, Gi0/2
                                              Gi0/4
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```

---

### SW1 – `show interfaces trunk`

```text
SW1#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Fa0/1       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/1       1,10,99

Port        Vlans allowed and active in management domain
Fa0/1       1,10,99

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1,10,99
```

---

### R1 – `show run`

```text
R1#show run
Building configuration...

Current configuration : 2646 bytes
!
! Last configuration change at 11:11:37 CET Wed Nov 26 2025
! NVRAM config last updated at 11:12:01 CET Wed Nov 26 2025
! NVRAM config last updated at 11:12:01 CET Wed Nov 26 2025
version 15.1
service timestamps debug datetime localtime show-timezone
service timestamps log datetime localtime show-timezone
no service password-encryption
!
hostname R1
!
boot-start-marker
boot system flash:c1841-advipservicesk9-mz.151-4.M12a.bin
boot-end-marker
!
!
logging buffered 16384 informational
!
aaa new-model
!
!
aaa authentication login VTY-LOCAL local
!
!
!
!
!
aaa session-id common
!
clock timezone CET 1 0
clock summer-time CEST recurring
dot11 syslog
ip source-route
!
!
ip dhcp excluded-address 192.168.10.1 192.168.10.9
!
ip dhcp pool HostIPs
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
!
!
!
ip cef
ip domain name R1.lab.internal
login block-for 120 attempts 3 within 60
login quiet-mode access-class 99
no ipv6 cef
!
multilink bundle-name authenticated
!
crypto pki token default removal timeout 0
!
!
!
!
license udi pid CISCO1841 sn FCZ151294FJ
username netadmin privilege 15 secret 5 $1$3MDO$/8ZZdkCIb2F7whOS5eNAw.
!
redundancy
!
!
ip ssh version 2
!
!
!
!
!
!
interface FastEthernet0/0
 no ip address
 ip nat inside
 ip virtual-reassembly in
 duplex auto
 speed auto
!
interface FastEthernet0/0.1
 encapsulation dot1Q 1 native
 ip address 10.0.0.2 255.255.255.252
!
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
interface FastEthernet0/0.99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0
!
interface FastEthernet0/1
 description WAN-To-ISP
 no ip address
 ip nat outside
 ip virtual-reassembly in
 shutdown
 duplex auto
 speed auto
!
ip forward-protocol nd
no ip http server
no ip http secure-server
!
!
ip nat inside source list 1 interface FastEthernet0/1 overload
ip route 0.0.0.0 0.0.0.0 10.0.0.1
!
!
logging 192.168.99.50
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.99.0 0.0.0.255
access-list 99 permit 192.168.99.50
!
!
!
!
snmp-server group GRLAB1 v3 priv read READONLY
snmp-server view READONLY iso included
snmp-server location Server Room A1
snmp-server contact nettops@example.com
!
!
!
!
control-plane
!
!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 privilege level 15
 login authentication VTY-LOCAL
 refuse-message Unauthorized access is prohibited.
 transport input ssh
line vty 5 15
 transport input none
!
scheduler allocate 20000 1000
ntp authentication-key 1 md5 0007030E105206035E731F 7
ntp authenticate
ntp trusted-key 1
ntp server 192.168.99.51 key 1
end
```

---

### SW1 – `show run`

```text
SW1#show run
Building configuration...

Current configuration : 7011 bytes
!
! Last configuration change at 11:13:30 CET Wed Nov 26 2025
! NVRAM config last updated at 11:13:46 CET Wed Nov 26 2025
!
version 12.2
no service pad
service timestamps debug datetime localtime show-timezone
service timestamps log datetime localtime show-timezone
no service password-encryption
!
hostname SW1
!
boot-start-marker
boot-end-marker
!
logging buffered 16834 informational
!
username netadmin privilege 15 secret 5 $1$8BaR$.L.hOd02WDi4LV6gBAh/M/
!
!
aaa new-model
!
!
aaa authentication login VTY-LOCAL local
!
!
!
aaa session-id common
clock timezone CET 1
clock summer-time CEST recurring
system mtu routing 1500
!
!
ip domain-name SW1.lab.internal
login block-for 120 attempts 3 within 60
login quiet-mode access-class 99
!
!
crypto pki trustpoint TP-self-signed-3986326016
 enrollment selfsigned
 subject-name cn=IOS-Self-Signed-Certificate-3986326016
 revocation-check none
 rsakeypair TP-self-signed-3986326016
!
!
crypto pki certificate chain TP-self-signed-3986326016
 certificate self-signed 01
  3082024C 308201B5 A0030201 02020101 300D0609 2A864886 F70D0101 04050030
  31312F30 2D060355 04031326 494F532D 53656C66 2D536967 6E65642D 43657274
  69666963 6174652D 33393836 33323630 3136301E 170D3933 30333031 30303032
  30395A17 0D323030 31303130 30303030 305A3031 312F302D 06035504 03132649
  4F532D53 656C662D 5369676E 65642D43 65727469 66696361 74652D33 39383633
  32363031 3630819F 300D0609 2A864886 F70D0101 01050003 818D0030 81890281
  8100C733 A98829FD 13D01DE1 82B63030 E57CAE70 E3CB8409 38981777 984C88ED
  6C9C4216 CF414139 078F6BD4 8C9FAF49 FB61944C 11455EB7 72F327AA A96FA125
  43496606 654887B0 E26D5DD3 CF18A8A1 B5EF79BD 985E0019 540668E2 30950C96
  234CF211 C765C2FB A2EC40D3 B64BB75A 78952826 82505CFE 3C3C9EDF F779F797
  60EF0203 010001A3 74307230 0F060355 1D130101 FF040530 030101FF 301F0603
  551D1104 18301682 14535731 2E535731 2E6C6162 2E696E74 65726E61 6C301F06
  03551D23 04183016 801478AE 33F59D1F 6730D811 C817BF9E 6269B33B 7FB6301D
  0603551D 0E041604 1478AE33 F59D1F67 30D811C8 17BF9E62 69B33B7F B6300D06
  092A8648 86F70D01 01040500 03818100 064AE213 3002B733 F93973D3 7CBA9EAA
  F67A610E CC10B1CE C07BEEB2 851D04D1 ED4DFD31 859BC5DD C5A5A9D6 31675C35
  872D535C FCA49A63 3FAF835E B358A97E 7150387A 2A91856B 6D35C134 4DEDA63D
  602E5CCA B5D8F802 49B15A4D 6544487E D66C0E71 E61F8689 832D0B5F E4E25E87
  6420C1CE 4FD19F5A 08D99043 03495F56
  quit
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
vlan internal allocation policy ascending
!
ip ssh version 2
!
!
interface FastEthernet0/1
 description Trunk to R1
 switchport trunk allowed vlan 1,10,99
 switchport mode trunk
!
interface FastEthernet0/2
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/3
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/4
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/5
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/6
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/7
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/8
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/9
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/10
 description User WorkStation
 switchport access vlan 10
 switchport mode access
 switchport port-security
 switchport port-security mac-address sticky
 spanning-tree portfast
!
interface FastEthernet0/11
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/12
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/13
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/14
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/15
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/16
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/17
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/18
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/19
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/20
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/21
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/22
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/23
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/24
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/25
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/26
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/27
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/28
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/29
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/30
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/31
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/32
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/33
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/34
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/35
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/36
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/37
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/38
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/39
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/40
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/41
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/42
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/43
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/44
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/45
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/46
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/47
 switchport access vlan 999
 shutdown
!
interface FastEthernet0/48
 switchport access vlan 999
 shutdown
!
interface GigabitEthernet0/1
 switchport access vlan 999
 shutdown
!
interface GigabitEthernet0/2
 switchport access vlan 999
 shutdown
!
interface GigabitEthernet0/3
 switchport mode access
!
interface GigabitEthernet0/4
 switchport access vlan 999
 shutdown
!
interface Vlan1
 no ip address
!
interface Vlan99
 ip address 192.168.99.2 255.255.255.0
!
ip default-gateway 192.168.99.1
ip http server
ip http secure-server
!
ip access-list standard MGMT_ONLY
 permit 192.168.99.0 0.0.0.255
!
logging 192.168.99.50
access-list 99 permit 192.168.99.50
snmp-server group GRLAB1 v3 priv read READONLY
snmp-server view READONLY iso included
snmp-server location Server Room A1
snmp-server contact nettops@example.com
!
line con 0
 logging synchronous
line vty 0 4
 access-class MGMT_ONLY in
 privilege level 15
 login authentication VTY-LOCAl
 refuse-message Unauthorized access is prohibited.
 transport input ssh
line vty 5 15
 transport input none
!
ntp authentication-key 1 md5 121A0D07060201017B7977 7
ntp.authenticate
ntp trusted-key 1
ntp server 192.168.99.51 key 1
end
```

---

## 6. Host System Details

### Ubuntu Host

- OS Version: Ubuntu
- Hardware:
  - Architecture: x86_64
  - CPU: Intel(R) Core(TM) i7-8565U CPU @ 1.80GHz
  - Cores / Threads: 4 cores, 8 threads (2 threads per core)
  - Max / Min Frequency: 4.60 GHz max, 0.40 GHz min
  - Virtualization: VT-x supported
  - Cache:
    - L1d: 128 KiB (4 instances)
    - L1i: 128 KiB (4 instances)
    - L2: 1 MiB (4 instances)
    - L3: 8 MiB (1 instance)

### Network Devices

| Device | Model       | OS Version                                  |
|--------|-------------|---------------------------------------------|
| R1     | Cisco 1841  | c1841-advipservicesk9-mz.151-4.M12a.bin    |
| SW1    | Cisco 2960  | IOS 12.2       |

## 7. Notes & Lessons Learned

- Learned how to properly structure and document a network engineering ticket in a real-world format.
- Identified how VLANs, trunk links, and SVIs form the foundation of a functional lab environment.
- Documenting configs takes longer than expected but is extremely useful.
- Clarified the relationship between logical (VLAN/IP) and physical (cabling/interfaces) topology.
- Ensured consistency between device configs (R1/SW1) and the documented topology.
- VLAN and trunking verification (`show vlan`, `show int trunk`) is now second nature.
- Learned that unused ports should be shut down and placed in a parking VLAN.
- Demonstrated DHCP, inter-VLAN routing, and management segmentation.
- Reinforced the value of “show” commands for verification.
- SNMPv3 configuration required careful group/user matching — good learning experience.
- Time sync (NTP) requires a proper server; without one the devices appear up but remain unsynchronized.

---

## Status
**Ticket 00: Completed and documented.**
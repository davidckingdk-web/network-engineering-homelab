# Ticket 02 – Routing & NAT

## 1. Summary of Work Completed

Configured R1 as the default gateway and edge router for the lab VLANs, with:
- Inter-VLAN routing for VLAN 10 (Users) and VLAN 99 (Management)
- A point-to-point transit network towards the ISP (10.0.0.0/30)
- A static default route pointing to the ISP
- NAT overload (PAT) so inside hosts in VLAN 10 (and optionally VLAN 99) can reach upstream networks via R1

All connectivity inside the lab and towards the simulated ISP gateway (10.0.0.1) has been verified. 
Internet pings (e.g. 8.8.8.8) fail as expected because there is no real upstream internet behind the ISP.

---

## 2. Lab Environment

**Devices involved:**
- R1 – Cisco 1841 router (IOS 15.1, `c1841-advipservicesk9-mz.151-4.M12a.bin`)
- SW1 – Cisco Catalyst switch (carries VLAN 10, 99 and native VLAN 1 up to R1 on a trunk)
- Ubuntu host – connected to SW1 access port in VLAN 10, using DHCP from R1
- ISP router – simulated upstream device on 10.0.0.0/30 (no real internet behind it)

**Physical connections (final design):**
- R1 FastEthernet0/0 → ISP router (transit network 10.0.0.0/30)
- R1 FastEthernet0/1 → SW1 GigabitEthernet0/3 (802.1Q trunk carrying VLAN 1, 10, 99)
- SW1 FastEthernet0/10 → Ubuntu host (access port in VLAN 10)

**Relevant networks:**
- VLAN 10 – Users: `192.168.10.0/24`  
  - Default gateway: `192.168.10.1` (R1)
- VLAN 99 – Management: `192.168.99.0/24`  
  - Default gateway: `192.168.99.1` (R1)
- Transit to ISP: `10.0.0.0/30`  
  - R1: `10.0.0.2`  
  - ISP: `10.0.0.1`

---

## 3. R1 – Final Configuration (Routing & NAT)

Only the relevant parts are included here for clarity.

```cisco
hostname R1
!
ip dhcp excluded-address 192.168.10.1 192.168.10.9
!
ip dhcp pool HostIPs
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
!
ip cef
ip domain name R1.lab.internal
!
interface FastEthernet0/0
 ip address 10.0.0.2 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
 duplex auto
 speed auto
!
interface FastEthernet0/1
 no ip address
 ip nat inside
 ip virtual-reassembly in
 duplex auto
 speed auto
!
interface FastEthernet0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
interface FastEthernet0/1.99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0
!
ip nat inside source list 1 interface FastEthernet0/0 overload
ip route 0.0.0.0 0.0.0.0 10.0.0.1
!
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.99.0 0.0.0.255
```

Notes:
- `Fa0/0` is now the WAN/transit interface towards the ISP and is the NAT outside interface.
- `Fa0/1` is the physical interface towards SW1 and carries subinterfaces for VLAN 10 and VLAN 99.
- DHCP for the Users VLAN (10) is provided directly on R1.

---

## 4. SW1 – Relevant Configuration

Only routing/NAT‑related parts (i.e., the uplink to R1 and VLANs) are shown.

```cisco
hostname SW1
!
interface GigabitEthernet0/3
 description Trunk to R1
 switchport trunk allowed vlan 1,10,99
 switchport mode trunk
!
interface FastEthernet0/10
 description User WorkStation
 switchport access vlan 10
 switchport mode access
 spanning-tree portfast
!
interface Vlan99
 ip address 192.168.99.2 255.255.255.0
!
ip default-gateway 192.168.99.1
```

This ensures:
- VLAN 10 and 99 are carried over the trunk to R1.
- R1 provides inter-VLAN routing.
- SW1 can reach R1 via the management SVI on VLAN 99.

---

## 5. Verification – Connectivity & Routing

### 5.1 R1 – Interfaces and Routing Table

```cisco
R1# show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            10.0.0.2        YES NVRAM  up                    up
FastEthernet0/1            unassigned      YES NVRAM  up                    up
FastEthernet0/1.10         192.168.10.1    YES NVRAM  up                    up
FastEthernet0/1.99         192.168.99.1    YES NVRAM  up                    up
NVI0                       10.0.0.2        YES unset  up                    up

R1# show ip route
Gateway of last resort is 10.0.0.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 10.0.0.1
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.0.0.0/30 is directly connected, FastEthernet0/0
L        10.0.0.2/32 is directly connected, FastEthernet0/0
      192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.10.0/24 is directly connected, FastEthernet0/1.10
L        192.168.10.1/32 is directly connected, FastEthernet0/1.10
      192.168.99.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.99.0/24 is directly connected, FastEthernet0/1.99
L        192.168.99.1/32 is directly connected, FastEthernet0/1.99
```

### 5.2 Host ↔ R1 Connectivity

From the Ubuntu host in VLAN 10:

```bash
# IP and routes on the host
$ ip addr
2: enp0s31f6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 84:2a:fd:3e:9d:15 brd ff:ff:ff:ff:ff:ff
    inet 192.168.10.10/24 brd 192.168.10.255 scope global dynamic noprefixroute enp0s31f6
    ...

$ ip route
default via 192.168.10.1 dev enp0s31f6 proto dhcp src 192.168.10.10 metric 20100 
192.168.10.0/24 dev enp0s31f6 proto kernel scope link src 192.168.10.10 metric 100 

# Ping default gateway (R1 VLAN 10)
$ ping -c 4 192.168.10.1
64 bytes from 192.168.10.1: icmp_seq=1 ttl=255 time=1.1 ms
64 bytes from 192.168.10.1: icmp_seq=2 ttl=255 time=1.9 ms
...

# Ping R1 management SVI (VLAN 99) via inter-VLAN routing
$ ping -c 4 192.168.99.1
64 bytes from 192.168.99.1: icmp_seq=1 ttl=255 time=1.2 ms
64 bytes from 192.168.99.1: icmp_seq=2 ttl=255 time=1.2 ms
...

# Attempt to ping internet (expected to fail – no real ISP)
$ ping -c 4 8.8.8.8
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss
```

This confirms:
- The host receives a DHCP address in VLAN 10 from R1.
- The host can reach both its default gateway and the management SVI via routing.
- Default route is in place, but real internet connectivity is not available (lab limitation, not a config issue).

### 5.3 NAT Verification

Example NAT translation on R1 while the host sends traffic towards the ISP side:

```cisco
R1# show ip nat translations
Pro  Inside global         Inside local          Outside local       Outside global
icmp 10.0.0.2:xxxx         192.168.10.10:xxxx    10.0.0.1:yy         10.0.0.1:yy
```

This confirms that:
- Inside traffic from VLAN 10 is being translated to the transit IP (10.0.0.2) using overload.
- R1 is correctly routing and NATing traffic between VLAN 10 and the ISP-facing network, even though there is no further upstream internet.

---

## 6. Notes & Lessons Learned

- Using a dedicated physical interface for WAN (Fa0/0) and a separate router-on-a-stick trunk (Fa0/1 with subinterfaces) keeps the edge design clear.
- It’s easy to forget to move NAT roles when changing topology; verifying `ip nat inside` / `ip nat outside` after cabling changes is critical.
- A default route to the ISP only becomes meaningful once the transit link is up and reachable.
- DHCP, default gateway, and basic pings from a host are quick ways to validate end-to-end connectivity.
- Lack of internet reachability in a lab is not always a configuration problem – document clearly when the ISP side is only simulated.

---

## 7. What This Demonstrates to an Employer

- Ability to design and implement basic edge routing for a small site using a separate WAN interface and a trunk towards the access layer.
- Understanding of VLANs, SVIs, default gateways, and static default routes.
- Practical experience configuring and verifying NAT (PAT) on Cisco IOS.
- Comfort using Linux commands (`ip addr`, `ip route`, `ping`) alongside Cisco IOS for end-to-end troubleshooting.
- Clear documentation of both configuration and validation steps in a ticket-style format that mirrors real NOC / MSP workflows.
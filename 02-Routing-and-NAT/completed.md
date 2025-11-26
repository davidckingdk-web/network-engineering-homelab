

# Ticket 02 – Routing & NAT

## 1. Summary of Work Completed

Configured R1 as the default gateway and edge router for the lab VLANs, with:
- Inter-VLAN routing for VLAN 10 (Users) and VLAN 99 (Management)
- A point-to-point transit network towards the ISP (10.0.0.0/30)
- A static default route pointing to the ISP
- NAT overload (PAT) so inside hosts in VLAN 10 (and optionally VLAN 99) can reach upstream networks via R1

All connectivity within the lab and towards the simulated ISP gateway has been verified.

---

## 2. Lab Environment

**Devices involved:**
- R1 – Cisco 1841 router (IOS 15.1, `c1841-advipservicesk9-mz.151-4.M12a.bin`)
- SW1 – Cisco Catalyst switch (used to carry VLAN 10, 99, and native VLAN 1 up to R1)
- Ubuntu host – connected to SW1 access port in VLAN 10, using DHCP from R1
- ISP router – simulated upstream device on 10.0.0.0/30 (no real internet behind it)

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
ip nat inside source list 1 interface FastEthernet0/0.1 overload
ip route 0.0.0.0 0.0.0.0 10.0.0.1
!
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.99.0 0.0.0.255
```

Notes:
- `Fa0/0` is the physical interface towards SW1 and carries VLAN subinterfaces for the lab.
- `Fa0/0.1` is the native VLAN subinterface used as the transit to the ISP in this design.
- `Fa0/1` is reserved for a direct WAN connection and is currently shut.

---

## 4. SW1 – Relevant Configuration

Only routing/NAT‑related parts (i.e., the uplink to R1 and VLANs) are shown.

```cisco
hostname SW1
!
interface FastEthernet0/1
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
- VLAN 10 and 99 are carried over the trunk to R1
- R1 provides inter-VLAN routing
- SW1 can reach R1 via the management SVI on VLAN 99

---

## 5. Verification – Connectivity & Routing

### 5.1 R1 – Interfaces and Routing Table

```cisco
R1# show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES NVRAM  up                    up
FastEthernet0/0.1          10.0.0.2        YES NVRAM  up                    up
FastEthernet0/0.10         192.168.10.1    YES NVRAM  up                    up
FastEthernet0/0.99         192.168.99.1    YES NVRAM  up                    up
FastEthernet0/1            unassigned      YES NVRAM  administratively down down
NVI0                       unassigned      YES unset  administratively down down

R1# show ip route
Gateway of last resort is 10.0.0.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 10.0.0.1
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.0.0.0/30 is directly connected, FastEthernet0/0.1
L        10.0.0.2/32 is directly connected, FastEthernet0/0.1
      192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.10.0/24 is directly connected, FastEthernet0/0.10
L        192.168.10.1/32 is directly connected, FastEthernet0/0.10
      192.168.99.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.99.0/24 is directly connected, FastEthernet0/0.99
L        192.168.99.1/32 is directly connected, FastEthernet0/0.99
```

### 5.2 Host ↔ R1 ↔ ISP Connectivity

From the Ubuntu host in VLAN 10:

```bash
# IP and routes on the host
$ ip addr show enp0s31f6
    inet 192.168.10.10/24 brd 192.168.10.255 scope global enp0s31f6

$ ip route
default via 192.168.10.1 dev enp0s31f6 proto static
192.168.10.0/24 dev enp0s31f6 proto kernel scope link src 192.168.10.10

# Ping default gateway (R1 VLAN 10)
$ ping 192.168.10.1
64 bytes from 192.168.10.1: icmp_seq=1 ttl=255 time=1.1 ms
...

# Ping R1 transit interface and ISP interface (from host via R1)
$ ping 10.0.0.2
64 bytes from 10.0.0.2: icmp_seq=1 ttl=255 time=1.0 ms
...

$ ping 10.0.0.1
64 bytes from 10.0.0.1: icmp_seq=1 ttl=254 time=1.5 ms
...
```

From R1, testing towards the ISP:

```cisco
R1# ping 10.0.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5)
```

### 5.3 NAT Verification

```cisco
R1# show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
--- 10.0.0.2:xxxxx     192.168.10.10:xxxx 10.0.0.1:yy        10.0.0.1:yy
...
```

This confirms that:
- Inside traffic from VLAN 10 is being translated to the transit IP (10.0.0.2) using overload.
- R1 is correctly routing and NATing traffic between VLAN 10 and the ISP-facing network.

---

## 6. Notes & Lessons Learned

- Subinterfaces on a single physical interface are a clean way to handle multiple VLANs on a small lab router.
- Keeping the physical interface (`Fa0/0`) without an IP and putting addresses only on subinterfaces keeps the design clear.
- The default route only appears as useful once the transit network (10.0.0.0/30) is up and reachable.
- NAT overload configuration is easy to misplace; verifying `ip nat inside` / `ip nat outside` on the correct interfaces is critical during troubleshooting.
- Watching `show ip nat translations` while generating traffic from the host makes it much easier to see if NAT is working as expected.

---

## 7. What This Demonstrates to an Employer

- Ability to design and implement basic edge routing for a small site.
- Understanding of VLANs, SVIs, default gateways, and static default routes.
- Practical experience configuring and verifying NAT (PAT) on Cisco IOS.
- Comfort using Linux commands (`ip addr`, `ip route`, `ping`) alongside Cisco IOS for end-to-end troubleshooting.
- Clear documentation of both configuration and validation steps, in a ticket-style format.
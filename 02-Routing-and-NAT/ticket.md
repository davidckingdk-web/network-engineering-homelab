# Ticket 02 – Edge Routing & NAT

**Role:** Junior Network Engineer  
**Priority:** High  
**Location:** Small Office Lab – Edge Router R1

---

## Background

The office has:

- R1 as the edge router  
- SW1 as the access switch  
- VLAN 10 for Users (192.168.10.0/24)  
- VLAN 99 for Management (192.168.99.0/24)  
- An upstream “ISP” router reachable via a /30 link on VLAN 1

We need consistent edge routing and NAT so internal users can reach upstream networks through R1.

---

## Requirements

### 1. Edge Routing (R1)

On **R1**:

- Ensure the link toward the ISP uses:
  - Subinterface: `FastEthernet0/0.1`
  - Encapsulation: `dot1Q 1 native`
  - IP: `10.0.0.2/30`
- Configure a **default route**:
  - `0.0.0.0/0` via `10.0.0.1`
- Confirm the routing table shows:
  - Connected routes: `192.168.10.0/24`, `192.168.99.0/24`, `10.0.0.0/30`
  - A static default route with next-hop `10.0.0.1`

### 2. NAT Design (R1)

On **R1**:

- Treat **VLAN 10 (192.168.10.0/24)** as inside LAN  
- Treat the ISP-facing subinterface (`Fa0/0.1`) as outside  
- Configure **PAT (NAT overload)** so all 192.168.10.0/24 hosts share the outside IP:
  - Create a standard ACL to match 192.168.10.0/24
  - `ip nat inside` on `Fa0/0.10`
  - `ip nat outside` on `Fa0/0.1`
  - `ip nat inside source list … interface FastEthernet0/0.1 overload`
- Remove or clean up any **old NAT statements or interfaces** that no longer match the current topology (e.g. unused `FastEthernet0/1`).

### 3. Verification

On **R1**:

- `show ip interface brief`  
- `show ip route`  
- `show ip nat translations`  
- `show running-config | include ip nat`

From the **host in VLAN 10**:

- Confirm you receive an IP in `192.168.10.0/24` (via DHCP from Ticket 03)  
- Ping `192.168.10.1` (default gateway) – must succeed  
- Ping `10.0.0.1` (ISP router) – must succeed  
- Ping a “simulated internet” IP, e.g. `203.0.113.1` – even if it fails, confirm that:
  - `show ip nat translations` on R1 shows dynamic translation entries for the host

---

## Completion Criteria

- Default route to `10.0.0.1` present on R1  
- VLAN 10 and 99 still reachable from R1  
- Host in VLAN 10 can ping `192.168.10.1` and `10.0.0.1`  
- NAT configured cleanly using the **current** outside interface (`Fa0/0.1`)  
- `completed.md` created under `02-Routing-and-NAT/` with configs, show outputs, and notes

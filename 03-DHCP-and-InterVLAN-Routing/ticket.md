# Ticket 03 – DHCP & Inter-VLAN Routing

## 1. Ticket Description (from “Network Lead”)

We’ve brought up basic VLANs and routing on R1. Now we need to make sure that:

- Hosts in VLAN 10 receive IP settings automatically via DHCP from R1
- Inter-VLAN routing works between VLAN 10 (Users) and VLAN 99 (Management)
- The default gateway and DNS settings are correct on the client
- Connectivity from the client to management and transit networks is verified

Document the final configuration and the verification steps as if this were a real NOC / MSP ticket.

---

## 2. Current State

- **R1**:
  - Subinterfaces:
    - `Fa0/0.1` – `10.0.0.2/30` (native VLAN / transit)
    - `Fa0/0.10` – `192.168.10.1/24` (VLAN 10 – Users)
    - `Fa0/0.99` – `192.168.99.1/24` (VLAN 99 – Management)
  - DHCP pool for VLAN 10 already configured:
    - `network 192.168.10.0 255.255.255.0`
    - `default-router 192.168.10.1`
    - `dns-server 8.8.8.8`
- **SW1**:
  - `Fa0/1` trunk to R1 (1, 10, 99)
  - `Fa0/10` access in VLAN 10 (Ubuntu host)
  - `Vlan99` SVI with `192.168.99.2/24`
  - Default gateway should be `192.168.99.1`
- **Host (Ubuntu)**:
  - Connected to VLAN 10
  - Should receive DHCP from R1

---

## 3. Requirements

### 3.1 DHCP for VLAN 10
- Confirm R1 DHCP pool is correct.
- Confirm excluded addresses (e.g., `.1–.9`).
- Verify host actually receives DHCP.

### 3.2 Inter-VLAN Routing
- Routing must work between:
  - VLAN 10 → VLAN 99
  - VLAN 99 → VLAN 10
- Confirm SW1 gateway = `192.168.99.1`.

### 3.3 Client Verification
Verify on Ubuntu:
- IP = `192.168.10.x/24`
- Gateway = `192.168.10.1`
- DNS = `8.8.8.8`

### 3.4 Connectivity Tests
From Ubuntu:
- Ping `192.168.10.1`
- Ping `192.168.99.1`
- Ping `192.168.99.2`
- Ping `10.0.0.2`

From R1:
- Ping host in VLAN 10.

---

## 4. Acceptance Criteria

- [ ] DHCP lease visible in `show ip dhcp binding`
- [ ] Host in VLAN 10 receives IP, gateway, DNS
- [ ] Pings succeed (VLAN 10 ↔ VLAN 99)
- [ ] Pings succeed host ↔ R1
- [ ] All verification added to `completed.md`

---

## 5. Notes for Portfolio

In `completed.md`, highlight:
- DHCP pool config
- Inter-VLAN routing on subinterfaces
- Before/after DHCP behavior
- Troubleshooting steps used (`show ip dhcp binding`, `ip addr`, `ip route`, `ping`, etc.)

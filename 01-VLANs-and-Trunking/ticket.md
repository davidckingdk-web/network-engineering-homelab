# Ticket 01 – VLANs and Trunking Between R1 and SW1

## 1. Ticket Summary

Set up VLANs and 802.1Q trunking between the router (R1) and the switch (SW1) to support a routed management VLAN and a user VLAN, matching a small-office / lab environment.

This ticket assumes the physical and basic lab setup from **Ticket 00 – Lab Topology & Setup** is already in place.

---

## 2. Business / Lab Objective

Provide a clean Layer 2 design that:
- Separates user devices from management traffic.
- Uses a single physical link between R1 and SW1 for multiple VLANs (router-on-a-stick).
- Prepares the network for DHCP, security and monitoring tickets later in the roadmap.

This fits into **Phase 1 – Core Networking** of the overall home lab roadmap.

---

## 3. Requirements

### VLAN Design (SW1)

- Create the following VLANs on SW1:
  - VLAN 10 – `Users`
  - VLAN 99 – `Management`
  - VLAN 999 – `Parking`

- Ensure:
  - VLAN 10 is used for end-user devices (e.g. Ubuntu host).
  - VLAN 99 is used for management (SVI on SW1, R1 subinterface, SNMP/syslog later).
  - VLAN 999 is used as a “parking” VLAN for all unused access ports.

### Trunk Configuration (R1 ↔ SW1)

- On SW1:
  - Use `FastEthernet0/1` as a trunk towards R1.
  - Allowed VLANs on the trunk: `1,10,99`.
  - Native VLAN: `1` (default).

- On R1:
  - Use `FastEthernet0/0` as the physical interface towards SW1.
  - Create 802.1Q subinterfaces:
    - `Fa0/0.1` – native VLAN 1, used as a transit/WAN-style link (e.g. `10.0.0.2/30`).
    - `Fa0/0.10` – VLAN 10, user subnet `192.168.10.0/24`.
    - `Fa0/0.99` – VLAN 99, management subnet `192.168.99.0/24`.

  - IP addressing (example):
    - `Fa0/0.1` → `10.0.0.2/30`
    - `Fa0/0.10` → `192.168.10.1/24`
    - `Fa0/0.99` → `192.168.99.1/24`

### Access Ports (SW1)

- Place the Ubuntu host port into VLAN 10:
  - `Fa0/10` → access, VLAN 10, `spanning-tree portfast` enabled.

- Place **all other unused access ports** into VLAN 999 and shut them down:
  - `Fa0/2–Fa0/9`, `Fa0/11–Fa0/48`, `Gi0/1`, `Gi0/2`, `Gi0/4` → access VLAN 999 + `shutdown`.

- Keep `Gi0/3` free as a flexible lab port (can be left in VLAN 1 or reassigned later).

### Management SVI (SW1)

- Configure a management SVI:
  - `Vlan99` → `192.168.99.2/24`
- Set the default gateway on SW1:
  - `ip default-gateway 192.168.99.1` (R1’s VLAN 99 address).

---

## 4. Acceptance Criteria

The ticket is considered **completed** when:

1. **VLANs exist on SW1**  
   - `show vlan brief` shows VLANs 10, 99, and 999 with the correct ports assigned.

2. **Trunk is up and carrying VLANs**  
   - `show interfaces trunk` on SW1 shows:
     - `Fa0/1` as trunking
     - Allowed and active VLANs: `1,10,99`.

3. **R1 subinterfaces are up/up**  
   - `show ip interface brief` on R1 shows:
     - `Fa0/0.1` → `10.0.0.2` (up/up)
     - `Fa0/0.10` → `192.168.10.1` (up/up)
     - `Fa0/0.99` → `192.168.99.1` (up/up)

4. **Host connectivity works**
   - From the Ubuntu host in VLAN 10:
     - Can ping `192.168.10.1` (R1 VLAN 10 gateway).
     - Can ping `192.168.99.2` (SW1 management SVI).
   - From R1:
     - Can ping `192.168.10.10` 
     - Can ping `192.168.99.2`.

5. **Parking VLAN applied**
   - All unused access ports show:
     - `switchport access vlan 999`
     - `shutdown`

---

## 5. Commands to Capture for Documentation

When completing this ticket, plan to copy the output of:

- On **SW1**:
  - `show vlan brief`
  - `show interfaces trunk`
  - `show ip interface brief`
  - Relevant interface configs from `show running-config`:
    - `interface FastEthernet0/1`
    - `interface FastEthernet0/10`
    - `interface Vlan99`
    - A sample “parking” port (e.g. `interface FastEthernet0/2`)

- On **R1**:
  - `show ip interface brief`
  - `show running-config` sections for:
    - `interface FastEthernet0/0`
    - `interface FastEthernet0/0.1`
    - `interface FastEthernet0/0.10`
    - `interface FastEthernet0/0.99`

These will later go into `completed.md` for Ticket 01.

---

## 6. Links to Other Tickets

- **Ticket 00 – Lab Topology & Setup**  
  Provides the physical diagram, base addressing, and hardware overview used here.

- **Future Tickets**  
  - Ticket 02 – Routing and NAT
  - Ticket 03 – DHCP and Inter-VLAN Routing  

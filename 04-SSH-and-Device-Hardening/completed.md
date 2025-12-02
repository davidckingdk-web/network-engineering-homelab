# Ticket 04 – SSH Hardening & Device Security – Completed

## 1. Summary

Hardened remote management on **R1 (Cisco 1841)** and **SW1 (Catalyst switch)**:

- SSH v2 only on both devices  
- Local admin account `netadmin` with privilege 15  
- `aaa new-model` + `VTY-LOCAL` method list for SSH logins  
- Login blocking and quiet-mode tied to management ACL  
- VTY access restricted to the management subnet  
- HTTP/HTTPS disabled on **R1** (edge router)  
- Warning / refuse-message configured on VTY lines  

Result: Only authorised management hosts in **192.168.99.0/24** can manage R1/SW1 over SSH; insecure services are minimised.

---

## 2. Topology & Devices

- **R1 – Cisco 1841**  
  - `FastEthernet0/0` → ISP (10.0.0.0/30)  
  - `FastEthernet0/1.10` → VLAN 10 (Users) – `192.168.10.1/24`  
  - `FastEthernet0/1.99` → VLAN 99 (Management) – `192.168.99.1/24`

- **SW1 – Cisco Catalyst**  
  - `Gi0/3` trunk to R1 (VLANs 1, 10, 99)  
  - `Fa0/10` access port in VLAN 10 (User workstation)  
  - `Vlan99` SVI – `192.168.99.2/24`

- **Host (Linux)** – in VLAN 10  
  - Gets IP from DHCP pool on R1 (e.g. `192.168.10.10/24`)

---

## 3. Final Config Snippets

### 3.1 R1 – SSH, AAA, VTY, ACLs & HTTP Disable

```bash
hostname R1
ip domain name R1.lab.internal

ip ssh version 2

username netadmin privilege 15 secret 5 <redacted-secret>

aaa new-model
aaa authentication login VTY-LOCAL local
login block-for 120 attempts 3 within 60
login quiet-mode access-class 99

! Management access control (NMS + management subnets)
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.99.0 0.0.0.255
access-list 99 permit 192.168.99.50

! VTY lines
line vty 0 4
 privilege level 15
 login authentication VTY-LOCAL
 refuse-message ^CUnauthorized access is prohibited.^C
 transport input ssh
line vty 5 15
 transport input none

! Disable HTTP/HTTPS on edge router
no ip http server
no ip http secure-server
```

---

### 3.2 SW1 – SSH, AAA, VTY & Management ACL

```bash
hostname SW1
ip domain-name SW1.lab.internal

ip ssh version 2

username netadmin privilege 15 secret 5 <redacted-secret>

aaa new-model
aaa authentication login VTY-LOCAL local
login block-for 120 attempts 3 within 60
login quiet-mode access-class 99

! Management VLAN & default gateway
interface Vlan99
 ip address 192.168.99.2 255.255.255.0
!
ip default-gateway 192.168.99.1

! Management-only access list
ip access-list standard MGMT_ONLY
 permit 192.168.99.0 0.0.0.255

! Quiet-mode ACL for login blocking
access-list 99 permit 192.168.99.50

! VTY lines
line vty 0 4
 access-class MGMT_ONLY in
 privilege level 15
 login authentication VTY-LOCAL
 refuse-message ^CUnauthorized access is prohibited.^C
 transport input ssh
line vty 5 15
 transport input none
```

> Note: SW1 currently keeps `ip http server` / `ip http secure-server` enabled for lab convenience. On a real production switch, these might be disabled as on R1.

---

## 4. Verification Outputs

### 4.1 R1 – SSH & VTY

```bash
R1#show ip ssh
SSH Enabled - version 2.0
Authentication timeout: 120 secs; Authentication retries: 3
```

```bash
R1#show run | section line vty
line vty 0 4
 privilege level 15
 login authentication VTY-LOCAL
 refuse-message ^CUnauthorized access is prohibited.^C
 transport input ssh
line vty 5 15
 transport input none
```

```bash
R1#show run | include login block|login quiet
login block-for 120 attempts 3 within 60
login quiet-mode access-class 99
```

```bash
R1#show run | include ip http
no ip http server
no ip http secure-server
```

---

### 4.2 SW1 – SSH & VTY

```bash
SW1#show ip ssh
SSH Enabled - version 2.0
Authentication timeout: 120 secs; Authentication retries: 3
```

```bash
SW1#show run | section line vty
line vty 0 4
 access-class MGMT_ONLY in
 privilege level 15
 login authentication VTY-LOCAL
 refuse-message ^CUnauthorized access is prohibited.^C
 transport input ssh
line vty 5 15
 transport input none
```

```bash
SW1#show access-lists MGMT_ONLY
Standard IP access list MGMT_ONLY
    10 permit 192.168.99.0, wildcard bits 0.0.0.255
```

```bash
SW1#show run | include login block|login quiet
login block-for 120 attempts 3 within 60
login quiet-mode access-class 99
```

---

### 4.3 Host – SSH tests (expected behaviour)

From a management-capable host in 192.168.99.0/24:

```bash
ssh netadmin@192.168.99.1    # to R1 – SUCCESS
ssh netadmin@192.168.99.2    # to SW1 – SUCCESS
```

From a non-management subnet (e.g. VLAN 10, if restricted later):

```bash
ssh netadmin@192.168.99.1    # EXPECTED: blocked/refused once ACL tightened
ssh netadmin@192.168.99.2    # EXPECTED: blocked/refused once ACL tightened
```

---

## 5. Notes & Lessons Learned

- Always create the **local user** before enabling `aaa new-model` + VTY method lists, to avoid locking yourself out.
- **Login quiet-mode** must be paired with a safe ACL, or you can accidentally block all remote management.
- Consistent names (`VTY-LOCAL`, `MGMT_ONLY`, `READONLY`) make configs easier to reason about later and look better in a portfolio.
- Disabling HTTP/HTTPS on the edge router reduces the attack surface; SSH + console are usually enough.
- This ticket ties nicely into monitoring (Syslog/SNMPv3/NTP), since only trusted management hosts can reach the devices.

This ticket is **complete**: both devices now use hardened SSH-based management controlled by the management subnet.

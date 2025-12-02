

# Ticket 04 – SSH Hardening & Device Security

## 1. Context
A new small-office lab network has been built with:
- **R1** – Cisco 1841 acting as the edge router, default gateway and DHCP server
- **SW1** – Cisco Catalyst switch providing VLANs and trunking
- **Host** – Linux workstation in **VLAN 10 (Users)**
- **Management VLAN:** 192.168.99.0/24 (SVI on SW1 and subinterface on R1)

The company now wants to standardise **secure remote access** and **basic device-hardening** on all network devices.

This ticket focuses on **SSH, AAA, management access control, and disabling insecure services** on both R1 and SW1.

---

## 2. Objective
Harden R1 and SW1 so that:
- Only **SSH v2** is used for remote admin
- Only **authorised management hosts** in **192.168.99.0/24** can access VTY lines
- Local user **`netadmin`** is used for SSH login with **privilege 15**
- Login attempts are rate-limited to protect against brute-force attacks
- Insecure services (HTTP/HTTPS on R1) are disabled

At the end, you should be able to SSH from the Linux host (or another management host in VLAN 99) to both devices using the `netadmin` account, while other sources are blocked.

---

## 3. Devices in Scope
- **R1** – Cisco 1841
- **SW1** – Cisco Catalyst switch

---

## 4. Requirements

### 4.1 Hostname & Domain Name
- Confirm the following:
  - **R1 hostname:** `R1`
  - **R1 domain-name:** `R1.lab.internal`
  - **SW1 hostname:** `SW1`
  - **SW1 domain-name:** `SW1.lab.internal`

> If anything is different, update it to match.

### 4.2 SSH v2 Only + RSA Keys
On **both R1 and SW1**:
- Generate **RSA keys** (minimum 2048 bits if supported on the platform)
- Enforce **SSH version 2 only**

### 4.3 Local Admin Account
On **both R1 and SW1**:
- Create (or verify) a local user:
  - **Username:** `netadmin`
  - **Privilege:** 15
  - **Password:** use a **secret** (type 5 or 9), not clear-text

### 4.4 AAA for VTY Login
On **both R1 and SW1**:
- Enable **AAA** (if not already enabled)
- Configure **VTY login** to use **local** authentication via an AAA method list, e.g.:
  - `aaa authentication login VTY-LOCAL local`
  - Apply this method list to **VTY 0–4** (and additional lines if present)

### 4.5 Login Protection (Brute Force Mitigation)
On **both R1 and SW1**:
- Configure the device to **block logins** after repeated failures, for example:
  - Block for **120 seconds** after **3 failed attempts** within **60 seconds**
- Ensure the configuration also uses **quiet-mode access-class** tied to your management ACL, so that only trusted hosts are allowed when quiet mode is active.

### 4.6 Restrict VTY Access to Management VLAN
On **both R1 and SW1**:
- Ensure there is a **standard ACL** that permits only **192.168.99.0/24** (management subnet). Example naming:
  - `ip access-list standard MGMT_ONLY`
- Apply this ACL as:
  - `access-class MGMT_ONLY in` on the VTY lines
- Confirm that **only hosts in 192.168.99.0/24** can initiate SSH sessions.

### 4.7 Disable Insecure Services
- On **R1** (edge router):
  - Ensure **HTTP** and **HTTPS** servers are **disabled**:
    - `no ip http server`
    - `no ip http secure-server`
- On **SW1**:
  - Decide whether to keep the web UI or disable it. For a secure lab, you can also disable HTTP/HTTPS the same way for consistency.

### 4.8 SSH Banner / Refuse-Message
On **both R1 and SW1**:
- Configure a clear but professional **unauthorised access warning** using either:
  - A **login banner** (`banner login`) **or**
  - `refuse-message` under the VTY lines

Example wording (you can adapt):
> "Unauthorized access is prohibited. All activity may be monitored and reported."

---

## 5. Acceptance Criteria
To consider this ticket **DONE**, the following must be true:

1. **SSH v2 only** is enabled on R1 and SW1 (`show ip ssh`).
2. A **local user `netadmin` with privilege 15** exists on both devices (`show run | include username`).
3. **AAA new-model** is enabled and **VTY lines** use the `VTY-LOCAL` method list.
4. **Login block** / quiet-mode configuration is present (`show run | include login block|login quiet`).
5. **VTY lines** are restricted by the `MGMT_ONLY` ACL (or equivalent) and only allow management subnet 192.168.99.0/24.
6. **HTTP/HTTPS** is disabled on R1 (and optionally SW1) – verified via `show run | include ip http`.
7. SSH from a **management host in VLAN 99** to:
   - `R1` (192.168.99.1)
   - `SW1` (192.168.99.2)
   is **successful** using `netadmin`.
8. SSH from a **non-management subnet** (for example, temporarily from VLAN 10 if you test it that way) is **denied**.

---

## 6. Verification Commands (Suggested)

On **R1 and SW1**:

```bash
show ip ssh
show run | section line vty
show run | include aaa authentication login
show run | include username netadmin
show access-lists MGMT_ONLY
show line vty 0 4
show run | include login block|login quiet
show run | include ip http
```

From a **management host** in VLAN 99:

```bash
ssh netadmin@192.168.99.1   # SSH to R1
ssh netadmin@192.168.99.2   # SSH to SW1
```

(Optionally, also try SSH from a non-management subnet and confirm it fails.)

---

## 7. Documentation to Capture in `completed.md`
When you complete this ticket, capture at least:

- Final SSH / AAA / VTY sections from **R1** and **SW1** (`show run` snippets)
- `show ip ssh` outputs from both devices
- `show access-lists MGMT_ONLY` from both devices
- Sample SSH session transcript from the management host (success)
- Optional: failed SSH attempt from a non-authorised source
- Any issues you hit while configuring crypto, AAA, or access-classes

These will go into `04-SSH-and-Device-Hardening/completed.md`.
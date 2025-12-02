# Ticket 05 – Syslog and Monitoring

## 1. Summary

Configure centralized logging for the lab so that both the edge router (R1) and the switch (SW1) send syslog messages to a remote logging server, while still keeping useful local logs on each device.

This is written like a real NOC / MSP ticket and focuses on operational visibility and troubleshooting.

---

## 2. Current State

- R1 and SW1 are already:
  - Reachable via SSH using local AAA.
  - Using VLAN 10 for users and VLAN 99 for management.
  - Using 192.168.99.0/24 as the management subnet.
- There is a planned syslog server at:
  - IP: `192.168.99.50`
  - Network: `192.168.99.0/24`

Devices:
- **R1** (router-on-a-stick)
  - G0/0 (FastEthernet0/0) → ISP (10.0.0.0/30)
  - F0/1.10 → VLAN 10 (192.168.10.0/24)
  - F0/1.99 → VLAN 99 (192.168.99.0/24)
- **SW1**
  - Trunk to R1 on Gi0/3 (VLANs 1,10,99)
  - User access port on Fa0/10 (VLAN 10)
  - Management SVI: VLAN 99 → 192.168.99.2/24

---

## 3. Requirements

1. Enable syslog on R1 and SW1 with:
   - Local logging buffer of at least **16 KB** at **informational** level.
   - Timestamps on all log messages (with local time and timezone).
2. Configure both devices to send syslog to the remote server:
   - Destination: `192.168.99.50`
   - Transport: UDP 514 (default).
3. Ensure logs include:
   - Configuration changes.
   - Interface up/down events.
4. Verify that:
   - R1 and SW1 are actively logging locally.
   - Both devices show that they are trying to send logs to `192.168.99.50`.

*(If a real syslog daemon is not yet running on 192.168.99.50, verification can focus on device-side status and counters.)*

---

## 4. Constraints & Considerations

- Lab is running on real hardware (Cisco 1841 + managed switch + Linux host).
- NTP may not be fully functional yet, so timestamps might not be perfectly accurate, but configuration should still assume NTP usage.
- Management subnet is `192.168.99.0/24`, and all management-plane tools (SSH, syslog, SNMP, NTP) should use this network when possible.

---

## 5. Tasks

**On R1**
1. Enable detailed timestamps for logs and debug output:
   - Use local time and timezone.
2. Configure local logging:
   - Enable logging to an internal buffer of at least 16 KB.
   - Set logging level to informational.
3. Configure remote syslog:
   - Add `192.168.99.50` as a syslog host.
   - Ensure that R1 uses its management-facing interface (VLAN 99) as source for logs if supported.
4. Generate a couple of log messages (e.g., by doing a small config change) and confirm they appear in the local log buffer and are being sent to 192.168.99.50.

**On SW1**
5. Enable detailed timestamps for logs and debug output:
   - Use local time and timezone.
6. Configure local logging:
   - Enable logging to an internal buffer of at least 16 KB.
   - Set logging level to informational.
7. Configure remote syslog:
   - Add `192.168.99.50` as a syslog host.
8. Generate some log messages (e.g., shut/no shut on an interface, or VLAN change) and confirm they appear in the local log buffer and are being sent to 192.168.99.50.

**Verification from Linux host (optional if syslog daemon is running)**
9. From the Linux host in VLAN 10:
   - Confirm reachability to `192.168.99.50` and to R1/SW1.
   - Optionally, use tools like `tcpdump` to confirm that syslog (UDP/514) traffic is arriving at `192.168.99.50`.

---

## 6. Acceptance Criteria

- **R1**
  - `show logging` displays:
    - Logging to `192.168.99.50` on UDP/514.
    - A local log buffer configured at informational level.
  - `service timestamps` is configured for both debug and log messages with local time and timezone.
- **SW1**
  - `show logging` displays:
    - Logging to `192.168.99.50` on UDP/514.
    - A local log buffer configured at informational level.
  - `service timestamps` is configured for both debug and log messages with local time and timezone.
- (If syslog server is active) Linux host shows incoming syslog events from both R1 and SW1.

---

## 7. Devices and IP Reference

- **R1**
  - Hostname: `R1`
  - WAN: `FastEthernet0/0` → `10.0.0.2/30` (to ISP 10.0.0.1)
  - VLAN 10: `FastEthernet0/1.10` → `192.168.10.1/24`
  - VLAN 99: `FastEthernet0/1.99` → `192.168.99.1/24`

- **SW1**
  - Hostname: `SW1`
  - Trunk to R1: `Gi0/3` (VLANs 1,10,99)
  - User access: `Fa0/10` (VLAN 10)
  - Management SVI: `VLAN 99` → `192.168.99.2/24`

- **Syslog / NMS Server (planned)**
  - IP: `192.168.99.50`
  - Subnet: `192.168.99.0/24`

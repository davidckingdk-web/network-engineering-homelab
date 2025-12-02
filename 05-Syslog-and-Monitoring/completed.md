# Ticket 05 – Syslog and Centralized Monitoring (Completed)

## 1. Overview

This ticket configures centralized syslog for the lab:

- R1 and SW1 send logs to a remote syslog server at **192.168.99.50**.
- Local logging is still enabled (buffer) for on-box troubleshooting.
- Logs include timestamps with local time and timezone for easier correlation.

Result: An NMS / logging server on the management VLAN (192.168.99.0/24) can collect and analyze events from both devices.

---

## 2. Devices & IPs

- **R1** (Router)
  - Management / transit: `192.168.99.1/24` (Fa0/1.99)
  - Users VLAN: `192.168.10.1/24` (Fa0/1.10)
- **SW1** (Layer 2 switch with SVI)
  - Management SVI: `192.168.99.2/24` (Vlan99)
- **Syslog server** (NMS)
  - IP: `192.168.99.50/24`
  - Receives UDP syslog on port **514**.

All syslog traffic stays inside VLAN 99 (management network).

---

## 3. Final Configurations

### 3.1 Global logging settings (common pattern)

On both devices, logging timestamps are already enabled globally:

```cisco
service timestamps debug datetime localtime show-timezone
service timestamps log datetime localtime show-timezone
```

### 3.2 R1 – Syslog configuration

```cisco
! Global logging behaviour
logging buffered 16384 informational
logging 192.168.99.50
!
! ACL permitting the syslog server (used elsewhere as MGMT ACL)
access-list 99 permit 192.168.99.50
```

Relevant interface and IP context on R1:

```cisco
interface FastEthernet0/1.99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0
```

This ensures R1 can reach `192.168.99.50` via VLAN 99 and send logs there.

### 3.3 SW1 – Syslog configuration

```cisco
! Global logging behaviour
logging buffered 16834 informational
logging 192.168.99.50
!
! ACL 99 used by other features, also permits the NMS IP
access-list 99 permit 192.168.99.50
```

Relevant SVI and default gateway on SW1:

```cisco
interface Vlan99
 ip address 192.168.99.2 255.255.255.0
!
ip default-gateway 192.168.99.1
```

SW1 uses R1 (192.168.99.1) as default gateway but only needs L2/L3 connectivity within VLAN 99 to reach the syslog server.

---

## 4. Verification Commands & Key Outputs

### 4.1 R1 – `show logging`

```cisco
R1#show logging
Syslog logging: enabled (0 messages dropped, 3 messages rate-limited, 0 flushes, 0 overruns, xml disabled, filtering disabled)

    Console logging: level debugging, 23 messages logged, xml disabled,
                     filtering disabled
    Monitor logging: level debugging, 0 messages logged, xml disabled,
                     filtering disabled
    Buffer logging:  level informational, 2 messages logged, xml disabled,
                     filtering disabled
    Exception Logging: size (4096 bytes)
    Trap logging: level informational, 27 message lines logged
        Logging to 192.168.99.50  (udp port 514, audit disabled,
              link up),
              2 message lines logged,
              0 message lines rate-limited,
              0 message lines dropped-by-MD,
              xml disabled, sequence number disabled
              filtering disabled

Log Buffer (16384 bytes):
*Nov 26 09:01:01 UTC: %SYS-5-CONFIG_I: Configured from console by console
*Nov 26 09:01:02 UTC: %SYS-6-LOGGINGHOST_STARTSTOP: Logging to host 192.168.99.50 port 514 started - CLI initiated
```

Key points:

- **Syslog logging: enabled** – feature is active.
- **Logging to 192.168.99.50 (udp port 514, link up)** – remote server is configured and reachable.
- Log buffer shows config changes and confirmation of syslog-host start.

### 4.2 SW1 – `show logging`

```cisco
SW1#show logging
Syslog logging: enabled (0 messages dropped, 0 messages rate-limited, 0 flushes, 0 overruns, xml disabled, filtering disabled)

    Console logging: level debugging, 70 messages logged, xml disabled,
                     filtering disabled
    Monitor logging: level debugging, 0 messages logged, xml disabled,
                     filtering disabled
    Buffer logging:  level informational, 2 messages logged, xml disabled,
                     filtering disabled
    Exception Logging: size (4096 bytes)
    Trap logging: level informational, 73 message lines logged
        Logging to 192.168.99.50  (udp port 514,  audit disabled,
              authentication disabled, encryption disabled, link up),
              2 message lines logged,
              0 message lines rate-limited,
              0 message lines dropped-by-MD,
              xml disabled, sequence number disabled
              filtering disabled

Log Buffer (16834 bytes):
*Mar  1 01:48:45 UTC: %SYS-5-CONFIG_I: Configured from console by console
*Mar  1 01:48:46 UTC: %SYS-6-LOGGINGHOST_STARTSTOP: Logging to host 192.168.99.50 Port 514 started - CLI initiated
```

Key points:

- SW1 is also **logging to 192.168.99.50 (udp port 514, link up)**.
- Local buffer logging is enabled for quick on-device checks.

### 4.3 Basic reachability checks

(optional but implied from previous tickets)

From R1:

```cisco
R1#ping 192.168.99.2
R1#ping 192.168.99.50
```

From SW1:

```cisco
SW1#ping 192.168.99.1
SW1#ping 192.168.99.50
```

From the syslog/NMS host in VLAN 99 you would also verify:

```bash
ping 192.168.99.1
ping 192.168.99.2
```

If a syslog daemon is running on the server, new config changes on R1/SW1 should immediately appear in the server logs.

---

## 5. Notes & Lessons Learned

- Always enable **timestamps** for logs (`service timestamps ...`) so events can be correlated across devices and systems.
- Use a **management VLAN** (here, VLAN 99) for control-plane services like SSH, syslog, NTP, and SNMP. This keeps user traffic and management traffic separated.
- Keep **local logging (buffered)** enabled in addition to remote syslog. If the syslog server is down, the router/switch still has recent logs for troubleshooting.
- When using an ACL (like **access-list 99**) with multiple features (SNMP, vty quiet-mode, etc.), be careful when editing it so you don’t accidentally lock out management or monitoring.
- On a real job, you would typically standardize syslog levels (e.g. informational vs warnings only) per device role to avoid flooding the syslog server.

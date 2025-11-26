# Network Engineering Home Lab

This repository documents my practical network engineering lab environment.

I’m using real Cisco routers, a managed switch, and Linux hosts to simulate real-world tasks:
edge routing, VLANs, DHCP, NAT, SSH hardening, syslog, SNMPv3, NTP, and more.  
Each task is written like a “ticket” you’d receive in a Network Operations / NOC / MSP environment.

---

## 🔧 Technologies & Skills Demonstrated

- VLANs, trunking, and SVI design
- Inter-VLAN routing
- Static routing and default routes
- DHCP server configuration and relay
- NAT overload (PAT) on Cisco IOS
- SSH hardening, local AAA, and role-based access
- Syslog centralization to a remote logging server
- SNMPv2c and SNMPv3 secure monitoring
- NTP client configuration (with authentication)
- Switchport security (sticky MAC, parking VLAN)
- IOS image management and TFTP upgrades
- Linux basics for networking (ip addr, ip route, tcpdump, etc.)
- Documentation, verification and troubleshooting notes

---

## 📁 Lab Structure

Each folder in this repo represents a real-world style ticket or theme.

Planned structure:

- `00-Topology-and-Setup/` – physical diagram, hardware list, environment notes
- `01-VLANs-and-Trunking/` – VLAN design, trunks, SVIs
- `02-Routing-and-NAT/` – default route, NAT, internet edge style config
- `03-DHCP-and-InterVLAN-Routing/` – DHCP scopes, helper address, host connectivity
- `04-SSH-and-Device-Hardening/` – AAA, SSH, login control
- `05-Syslog-and-Monitoring/` – logging to a central server
- `06-NTP-and-Time-Sync/` – NTP setup 
- `07-SNMPv3-Configuration/` – secure monitoring users/groups/views
- `08-Port-Security-and-Switch-Security/` – sticky MAC, parking VLANs, shutdown unused ports
- `99-Notes-and-Lessons/` – troubleshooting journal and lessons learned

Each ticket folder will contain:

- `ticket.md` – description of the task in real-world ticket format
- `verification.md` – show commands, pings, and outputs
- `screenshots/` – optional images from the lab

---

## 🧭 Roadmap

**Phase 1 – Core Networking **  
Build a solid foundation and document:

- VLANs, trunking, SVI design
- DHCP, default routes, NAT
- SSH, syslog, SNMPv3, NTP
- Switchport security and cleanup

**Phase 2 – Routing & Monitoring**

- Add dynamic routing (OSPF/EIGRP in the lab)
- Introduce STP, HSRP, and better monitoring (NetFlow/Wireshark captures)

**Phase 3 – Automation**

- Start with simple scripts and API calls
- Move toward Ansible-style config management

**Phase 4 – Simulated Projects**

- Design and document a “small business” network
- Build an MSP-style monitoring setup
- Create playbooks for common incidents


## 🧑‍💻 About Me

I recently passed the CCNA and am transitioning into network engineering from a non-IT background.

This repo is my way of proving:
- I can configure and troubleshoot real devices
- I understand documentation and ticket-based workflows
- I am serious about learning and improving every week



# Ticket 01 – VLANs & Trunking  
**Status:** Completed  
**Date:** (auto-generated during work)

---

## 🔧 Summary of Work Completed

- Configured VLANs: **10 (Users)**, **99 (Management)**, **999 (Parking)**  
- Created a trunk between **SW1 FastEthernet0/1 → R1 FastEthernet0/0**  
- Confirmed native VLAN 1, allowed VLANs 1,10,99  
- Assigned access ports correctly  
- Verified VLAN table and trunk operational state  
- Confirmed SVI for VLAN 99 comes up  
- Verified end-device connectivity via ping

---

## 📘 Key Commands Used

### On SW1
```
vlan 10
 name Users
vlan 99
 name Management
vlan 999
 name Parking

interface fa0/1
 description Trunk to R1
 switchport trunk allowed vlan 1,10,99
 switchport mode trunk
```

### On R1
```
interface fa0/0.1
 encapsulation dot1Q 1 native
 ip address 10.0.0.2 255.255.255.252

interface fa0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface fa0/0.99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0
```

---

## 🖥️ Verification Outputs

### SW1 – VLANs  
```
show vlan brief
```

### SW1 – Trunk  
```
show interfaces trunk
```

### R1 – Subinterfaces  
```
show ip interface brief
```

Everything showed correct VLAN states, trunking, and interface operational status.

---

## 🧠 Notes & Lessons Learned

- The trunk must explicitly allow all VLANs being used between devices.  
- Native VLAN mismatches cause STP blocking or warnings — always verify both ends.  
- Parking VLANs (like VLAN 999) are a best practice to avoid security issues on unused ports.  
- SVI for VLAN 99 must be **up/up** which requires at least one active port in that VLAN.  
- Verification commands are just as important as configuration — “trust but verify.”

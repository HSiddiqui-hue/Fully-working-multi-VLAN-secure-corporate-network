# Fully-working-multi-VLAN-secure-corporate-network
Designed and implemented a fully functional multi-VLAN corporate network in Cisco Packet Tracer to simulate a real small-to-medium enterprise environment. The network includes HR, IT, and Sales departments, each isolated with VLANs and secured using multiple Layer-2 and Layer-3 security features.

Phase 1 — Secure Multi-VLAN Corporate Network (Cisco Packet Tracer)

This project is a **core network lab** built in Cisco Packet Tracer to simulate a small/medium corporate environment with **multiple VLANs, inter-VLAN routing and layered security controls**.

It’s designed to showcase skills relevant to roles like:

- ICT Support Technician / Helpdesk
- Computer Systems & Network Technician
- Network / Systems Support Officer
- Junior Network / Systems Administrator

## Topology Overview

- 1 × Cisco Router (Router-on-a-Stick)
- 1 × Layer 2 Switch (2960)
- Multiple PCs in three departments:
  - **HR** (VLAN 10)
  - **IT** (VLAN 20)
  - **Sales** (VLAN 30)

Routing is done using **router sub-interfaces (802.1Q trunk)** and the switch handles **VLAN segmentation, port security, DHCP snooping and Dynamic ARP Inspection (DAI)**.

Packet Tracer file:

- `PHASE1-Core-Network.pkt`

## VLAN & IP Addressing Plan

| VLAN | Name  | Subnet           | Default Gateway  |
|------|-------|------------------|------------------|
| 10   | HR    | 192.168.10.0/24  | 192.168.10.1     |
| 20   | IT    | 192.168.20.0/24  | 192.168.20.1     |
| 30   | SALES | 192.168.30.0/24  | 192.168.30.1     |

Router sub-interfaces:

- `G0/0.10` → 192.168.10.1 /24 (encapsulation dot1Q 10)
- `G0/0.20` → 192.168.20.1 /24 (encapsulation dot1Q 20)
- `G0/0.30` → 192.168.30.1 /24 (encapsulation dot1Q 30)

Switch uplink to router:

- `Fa0/1` configured as an 802.1Q **trunk** carrying VLANs 10, 20, 30.

## Security Features Implemented

### 1. Inter-VLAN Routing with Access Control Lists (ACLs)

- Router configured for **router-on-a-stick**.
- **Extended ACL** applied to control traffic between departments.
- Example policy:
  - **IT → HR is blocked** (192.168.20.0/24 cannot access 192.168.10.0/24).
  - Other traffic is permitted as normal.

Result:

- HR can reach IT/Sales (for support).
- IT **cannot** directly reach HR segment → better data isolation.

### 2. Port Security (Sticky MAC)

- Enabled on access ports (e.g. HR ports `Fa0/3`, `Fa0/4`).
- **Sticky MAC** learns the first connected device and locks the port to that MAC.
- Violation mode: **shutdown**
  - If someone unplugs the HR PC and connects a rogue device:
    - Port goes to **secure-shutdown**
    - `show port-security interface` shows:
      - Last source MAC
      - Violation count

This simulates protection against **unauthorized physical access** in an office.

### 3. DHCP Server on Router + DHCP Snooping on Switch

- Router acts as central **DHCP server** for all VLANs:
  - Separate pools for HR, IT, and Sales
  - Excluded gateway IPs
- Switch configured with:
  - `ip dhcp snooping`
  - `ip dhcp snooping vlan 10,20,30`
  - Uplink `Fa0/1` marked as **trusted**
  - Access ports remain **untrusted**

Effect:

- Only the legitimate router DHCP server can hand out IP addresses.
- Any rogue DHCP server connected to an access port would be blocked.
- `show ip dhcp snooping binding` displays:
  - IP address
  - MAC address
  - VLAN
  - Interface

This binding table is later used by DAI.

### 4. Dynamic ARP Inspection (DAI)

- DAI enabled on VLANs **10, 20, 30**.
- Uplink port `Fa0/1` configured as **trusted** for ARP inspection.
- All access ports are **untrusted**.
- DAI uses the **DHCP Snooping binding table** to validate ARP packets.

Result:

- ARP packets with **spoofed IP/MAC information** are dropped.
- Protects against basic ARP-spoof / man-in-the-middle attempts inside the LAN.
- Verified via `show ip arp inspection`:
  - VLANs 10/20/30 in **Active** state
  - Forwarded / dropped counters visible for troubleshooting.

> Note: Packet Tracer PCs do not support full ARP CLI commands (`arp -s` etc.), so ARP spoofing was simulated using static IP/MAC changes on clients rather than raw ARP commands (a limitation of the simulator, not of the design).

## Testing & Verification Summary

Some of the key tests performed:

1. **DHCP & Connectivity**
   - All PCs set to DHCP and successfully received IPs in correct VLAN/subnet.
   - Verified with `ipconfig` and `ping` to default gateway.

2. **ACL Behaviour**
   - From IT PC → ping HR PC / HR gateway → **fails** (blocked by ACL).
   - From HR PC → ping IT gateway / Sales → **succeeds**.

3. **Port Security**
   - After sticky MAC learned, original PC was disconnected and a different device was connected to the same port.
   - Port went into **secure-shutdown** and traffic stopped until admin intervened.

4. **DHCP Snooping**
   - Confirmed DHCP bindings appear with:
     - Correct IP, MAC, VLAN, interface.
   - Tested renewals with `ipconfig /release` and `ipconfig /renew`.

5. **DAI**
   - Enabled on all three VLANs and linked to DHCP Snooping.
   - Verified operation with `show ip arp inspection` and abnormal client configurations.

## Files in This Repository

- `PHASE1-Core-Network.pkt`  
  Main Cisco Packet Tracer project file.

- `images/topology.png` (optional)  
  Screenshot of the logical topology.

- `images/port-security-violation.png` (optional)  
  Example of a port-security shutdown event.

- `images/dhcp-snooping-binding.png` (optional)  
  Output of `show ip dhcp snooping binding`.

- `images/dai-status.png` (optional)  
  Output of `show ip arp inspection`.

## How to Open & Use This Lab

1. Install **Cisco Packet Tracer** (same or newer major version used to create the file).
2. Clone or download this repository.
3. Open `PHASE1-Core-Network.pkt` in Packet Tracer.
4. Start the simulation:
   - Click on each PC → Desktop → IP Configuration (ensure DHCP).
   - Use Command Prompt on PCs to `ping` gateways and other VLANs.
   - On the router/switch, use:
     - `show ip interface brief`
     - `show vlan brief`
     - `show running-config`
     - `show port-security interface fa0/x`
     - `show ip dhcp snooping`
     - `show ip dhcp snooping binding`
     - `show ip arp inspection`

## 📚 What I Practiced / Learned

- Designing an IP and VLAN plan for a small corporate network.
- Configuring **router-on-a-stick** inter-VLAN routing.
- Implementing **extended ACLs** for segmentation between departments.
- Applying **Port Security** with sticky MAC and violation modes.
- Securing DHCP with **DHCP Snooping**.
- Hardening Layer 2 with **Dynamic ARP Inspection (DAI)**.
- Troubleshooting complex issues (DHCP failures, ACL behaviour, port security shutdowns).

## Future Enhancements (Phase 2 Ideas)

- Add a second switch and configure **trunking** + **EtherChannel**.
- Implement **SSH** for secure device management.
- Add **Syslog** and **SNMP** monitoring.
- Introduce **IP Source Guard** on access ports.
- Simulate remote access using a VPN/router WAN link.

## How This Relates to Real Jobs

This lab demonstrates hands-on capability in:

- Building and troubleshooting multi-VLAN networks
- Enforcing network security at **Layer 2 and Layer 3**
- Supporting users across different departments safely
- Applying Cisco best practices used by:
  - ICT Support Technicians
  - Systems Support Officers
  - Network / Systems Admins in SME environments

Feel free to open an issue or fork this repo if you’d like to extend the lab further.

# 🔄 Switches — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 03 | **Status:** ✅ Complete

---

## 1. Definition

A **network switch** is a device that operates at **Layer 2 (Data Link Layer)** of the OSI Model. It connects multiple devices within the **same local network (LAN)** and uses **MAC addresses** to intelligently deliver data frames only to the intended destination device.

Unlike a hub (which sends data to every device), a switch learns where each device is and sends data only to the right port — making networks faster, more efficient, and more secure.

> Think of a switch as a smart mail sorter inside a building — it knows exactly which room (port) each person (device) is in, so it delivers mail (frames) directly rather than sliding it under every door.

---

## 2. How It Works — Step by Step

```
Step 1 — Device A sends a frame to Device B
Step 2 — Frame arrives at the switch on Port 1
Step 3 — Switch reads the SOURCE MAC address of Device A
Step 4 — Switch records: "Device A is on Port 1" in its MAC address table
Step 5 — Switch reads the DESTINATION MAC address (Device B)
Step 6 — Switch checks its MAC address table for Device B
         ├── Found? → Send frame directly out of Device B's port (unicast)
         └── Not found? → Flood frame out ALL ports except Port 1 (broadcast)
Step 7 — Device B receives the frame and responds
Step 8 — Switch learns Device B's port from the response
Step 9 — Future traffic between A and B goes directly — no flooding needed
```

---

## 3. Core Switch Functions

### 3.1 MAC Address Table (CAM Table)
The switch's brain. It stores a mapping of MAC addresses to physical ports.

```
Port  |  MAC Address         |  VLAN
------|----------------------|------
  1   |  AA:BB:CC:DD:EE:01  |  10
  2   |  AA:BB:CC:DD:EE:02  |  10
  3   |  AA:BB:CC:DD:EE:03  |  20
  4   |  AA:BB:CC:DD:EE:04  |  20
```

Entries age out after a period of inactivity (default ~300 seconds) to keep the table current.

### 3.2 Frame Forwarding Modes

| Mode | How it works | Speed | Error checking |
|------|-------------|-------|----------------|
| **Store-and-forward** | Receives full frame, checks for errors, then forwards | Slower | Yes — most reliable |
| **Cut-through** | Reads destination MAC only, forwards immediately | Faster | No |
| **Fragment-free** | Reads first 64 bytes (catches most errors), then forwards | Middle ground | Partial |

### 3.3 VLANs — Virtual Local Area Networks
VLANs allow a single physical switch to be divided into multiple **isolated logical networks**.

```
Physical switch with 3 VLANs:

[Port 1,2,3]  →  VLAN 10 — Finance     (isolated)
[Port 4,5,6]  →  VLAN 20 — HR          (isolated)
[Port 7,8,9]  →  VLAN 30 — IT          (isolated)
```

Devices in different VLANs **cannot communicate** without going through a **Layer 3 router** (or Layer 3 switch). This is a key security segmentation tool.

**VLAN types:**
| Type | Description |
|------|-------------|
| **Data VLAN** | Carries user-generated traffic |
| **Voice VLAN** | Dedicated for VoIP phones (prioritizes low latency) |
| **Management VLAN** | Used for managing network devices remotely |
| **Native VLAN** | Untagged traffic on a trunk port (security risk if misconfigured) |

### 3.4 Trunk Ports vs Access Ports

| Port type | Purpose |
|-----------|---------|
| **Access port** | Connects to end devices (PC, printer) — carries traffic for ONE VLAN |
| **Trunk port** | Connects switches to each other or to routers — carries traffic for MULTIPLE VLANs using 802.1Q tagging |

### 3.5 Spanning Tree Protocol (STP)
In redundant network designs, multiple switches are connected to each other for failover. Without STP, this creates **broadcast loops** that crash the network.

STP solves this by:
1. Electing a **Root Bridge** (the central switch)
2. Calculating the best path to the root from every switch
3. **Blocking** redundant ports to prevent loops
4. **Unblocking** them automatically if the primary path fails

**STP port states:**
```
Blocking → Listening → Learning → Forwarding → Disabled
```

Modern versions: **RSTP** (Rapid STP — faster convergence) and **MSTP** (Multiple STP — per-VLAN instances).

### 3.6 Port Security
Switches can be configured to limit which MAC addresses are allowed on a port:
- **Maximum MAC addresses** per port
- **Violation actions:** Shutdown, restrict, or protect the port if an unknown MAC appears

---

## 4. Types of Switches

| Type | Description |
|------|-------------|
| **Unmanaged switch** | Plug-and-play, no configuration, home/small office use |
| **Managed switch** | Fully configurable — VLANs, STP, port security, monitoring |
| **Layer 2 switch** | Standard switch — MAC-based forwarding only |
| **Layer 3 switch** | Can route between VLANs using IP addresses (combines switch + router) |
| **PoE switch** | Power over Ethernet — powers devices like IP cameras and phones through the cable |
| **Core switch** | High-speed switch at the center of a large enterprise network |
| **Access switch** | Connects end-user devices to the network |
| **Distribution switch** | Aggregates access switches and connects them to the core |

---

## 5. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **MAC address** | 48-bit hardware address burned into every NIC — used for Layer 2 delivery |
| **CAM table** | Content Addressable Memory — another name for the MAC address table |
| **Frame** | Layer 2 data unit — contains source/destination MAC, data, and FCS |
| **FCS** | Frame Check Sequence — error detection value at end of frame |
| **Unicast** | Frame sent to one specific device |
| **Broadcast** | Frame sent to all devices on the network (FF:FF:FF:FF:FF:FF) |
| **Multicast** | Frame sent to a group of devices |
| **802.1Q** | IEEE standard for VLAN tagging on trunk ports |
| **STP** | Spanning Tree Protocol — prevents Layer 2 loops |
| **RSTP** | Rapid Spanning Tree Protocol — faster version of STP |
| **PoE** | Power over Ethernet — delivers electrical power via network cable |
| **Flood** | Sending a frame out all ports when destination MAC is unknown |

---

## 6. 🔐 SOC Analyst Relevance

Switches are a major attack surface in internal networks. As a SOC analyst, these are the threats to watch for:

| Attack | What happens | Detection method |
|--------|-------------|-----------------|
| **MAC Flooding** | Attacker sends thousands of fake MACs to fill the CAM table — switch enters "fail-open" mode and broadcasts all traffic (attacker can sniff everything) | Sudden spike in MAC addresses on a port, CAM table overflow alerts |
| **ARP Poisoning / Spoofing** | Attacker sends fake ARP replies linking their MAC to a legitimate IP — intercepts traffic (Man-in-the-Middle) | Duplicate IP-to-MAC mappings in ARP table, IDS alerts |
| **VLAN Hopping** | Attacker tricks the switch into giving access to a different VLAN — bypasses network segmentation | Switch logs, 802.1Q double-tagging attempts |
| **STP Attack** | Attacker injects a rogue switch claiming to be Root Bridge — redirects all traffic through their device | Unexpected Root Bridge election in STP logs |
| **Rogue Switch** | Unauthorized switch added to network — can create loops or intercept traffic | LLDP/CDP neighbor discovery anomalies |
| **Port stealing** | Attacker floods switch with frames using victim's MAC as source — hijacks victim's port mapping | Rapid MAC movement between ports in logs |

**Key logs and data sources for SOC:**
- Switch syslog (port up/down events, security violations)
- ARP table monitoring (unexpected IP-MAC changes)
- DHCP logs (cross-reference with ARP)
- NetFlow (unusual lateral movement traffic patterns)
- 802.1X authentication logs (unauthorized device connection attempts)

> **Real SOC scenario:** An alert fires showing the same MAC address appearing on two different switch ports within seconds. This is a strong indicator of MAC spoofing or port stealing — worth immediate investigation.

---

## 7. Common Interview Questions

**Q1. What is the difference between a switch and a hub?**
> A hub broadcasts all traffic to every connected device — every device sees all traffic, causing collisions and security issues. A switch learns MAC addresses and forwards frames only to the intended device — much faster and more secure.

**Q2. What is a MAC address table and how does a switch build it?**
> The MAC address table (CAM table) maps MAC addresses to switch ports. The switch builds it dynamically — whenever a frame arrives, it records the source MAC address and the port it came from. This allows future frames to be sent directly to the right port.

**Q3. What is a VLAN and why is it used?**
> A VLAN (Virtual LAN) logically segments a physical network into isolated groups. It's used for security (separating departments), performance (reducing broadcast domains), and flexibility (grouping devices regardless of physical location).

**Q4. What is the difference between an access port and a trunk port?**
> An access port connects to end devices and carries traffic for a single VLAN. A trunk port connects switches together (or a switch to a router) and carries traffic for multiple VLANs simultaneously using 802.1Q tagging.

**Q5. What is MAC flooding and how would you detect it as a SOC analyst?**
> MAC flooding is when an attacker sends thousands of frames with fake source MACs to overflow the switch's CAM table. When full, the switch broadcasts all frames — allowing the attacker to sniff traffic. Detection: monitor for sudden spikes in unique MAC addresses on a single port or CAM table overflow alerts in the SIEM.

**Q6. What is ARP poisoning?**
> ARP poisoning (or ARP spoofing) is when an attacker sends fake ARP replies associating their MAC address with a legitimate IP address. This redirects traffic meant for the victim through the attacker's device — enabling a Man-in-the-Middle attack. Detected by monitoring ARP tables for duplicate or rapidly changing IP-to-MAC mappings.

**Q7. How does STP prevent network loops?**
> STP elects a Root Bridge and calculates the best path from every switch to the root. Redundant paths are put into a Blocking state — they carry no traffic but are ready to activate if the primary path fails. This prevents the broadcast storms that would occur in a looped topology.

---

## 8. Quick Revision Summary

```
Switch operates at   →  Layer 2 (Data Link Layer)
Primary function     →  Forward frames within a LAN using MAC addresses
Addresses used       →  MAC addresses (physical/hardware)
Key table            →  MAC address table / CAM table
Frame types          →  Unicast, Broadcast, Multicast
VLAN standard        →  IEEE 802.1Q
Loop prevention      →  STP / RSTP / MSTP
Port types           →  Access (single VLAN) vs Trunk (multiple VLANs)

Key attacks to know:
  MAC Flooding    →  Overflow CAM table → switch broadcasts everything
  ARP Poisoning   →  Fake ARP replies → Man-in-the-Middle
  VLAN Hopping    →  Escape VLAN segmentation
  STP Attack      →  Become Root Bridge → redirect all traffic
```

---

*Previous topic → [Routers](./routers.md)*
*Next topic → [Firewalls](./firewalls.md)*

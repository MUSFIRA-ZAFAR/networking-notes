# 🌐 Routers — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 02 | **Status:** ✅ Complete

---

## 1. Definition

A **router** is a network device that operates at **Layer 3 (Network Layer)** of the OSI Model. Its primary job is to receive data packets and forward them toward their destination across multiple different networks using **IP addresses**.

Unlike a switch (which connects devices within the same network), a router connects **different networks** together — your home network to the internet, or one office branch to another.

> Think of a router as a postal sorting facility — it reads the destination address on every package (packet) and decides which route it should take to get there fastest.

---

## 2. How It Works — Step by Step

```
Step 1 — Packet arrives at the router's input interface
Step 2 — Router strips the Layer 2 frame (MAC header) to read the Layer 3 IP header
Step 3 — Router reads the destination IP address
Step 4 — Router looks up the destination in its Routing Table
Step 5 — Router selects the best path (lowest cost / shortest hop)
Step 6 — Router forwards the packet out of the correct output interface
Step 7 — A new Layer 2 frame is created for the next hop
Step 8 — Process repeats at every router until packet reaches destination
```

---

## 3. Core Router Functions

### 3.1 Packet Forwarding
The most fundamental job of a router. Every incoming packet is read, the destination IP is extracted, and the packet is forwarded out of the correct interface toward its destination.

### 3.2 Routing Table
The router's internal "map" of the network. It stores:
- **Destination network** — what network is reachable
- **Next hop** — the IP of the next router to send the packet to
- **Interface** — which physical port to send it out of
- **Metric** — the cost of the route (used to pick the best path)

Routes are added to the table in three ways:
| Method | Description |
|--------|-------------|
| **Directly connected** | Networks physically connected to the router's interfaces |
| **Static routes** | Manually configured by a network admin |
| **Dynamic routes** | Learned automatically via routing protocols (OSPF, BGP, RIP) |

### 3.3 NAT — Network Address Translation
NAT allows multiple devices on a private network to share a single public IP address.

```
Private side (your devices)       Public side (internet)
192.168.1.10  ─┐
192.168.1.11  ─┤──► Router (NAT) ──► 203.0.113.5 (your public IP)
192.168.1.12  ─┘
```

**Types of NAT:**
| Type | Description |
|------|-------------|
| **Static NAT** | One private IP maps permanently to one public IP |
| **Dynamic NAT** | Private IPs mapped to a pool of public IPs |
| **PAT (Port Address Translation)** | Many private IPs share one public IP using different port numbers — most common in homes |

### 3.4 ACLs — Access Control Lists
Routers can filter traffic using ACLs — rules that permit or deny packets based on:
- Source/destination IP address
- Protocol (TCP, UDP, ICMP)
- Port number

```
Example ACL rule:
DENY  TCP  192.168.1.0/24  →  any  port 23   (block Telnet)
PERMIT IP  192.168.1.0/24  →  any            (allow everything else)
```

### 3.5 Routing Protocols
Dynamic routing protocols allow routers to automatically discover networks and share routing information with each other.

| Protocol | Type | Best used for |
|----------|------|---------------|
| **RIP** (Routing Information Protocol) | Distance-vector | Small networks |
| **OSPF** (Open Shortest Path First) | Link-state | Large enterprise networks |
| **EIGRP** (Enhanced IGRP) | Hybrid | Cisco networks |
| **BGP** (Border Gateway Protocol) | Path-vector | Internet backbone, ISPs |

---

## 4. Types of Routers

| Type | Description |
|------|-------------|
| **Home/SOHO router** | Consumer-grade, combines router + switch + Wi-Fi access point |
| **Core router** | High-speed routers at the backbone of the internet |
| **Edge router** | Sits at the boundary between an organization's network and the internet |
| **Virtual router** | Software-based router running in a VM or cloud environment |
| **Wireless router** | Router with built-in Wi-Fi capability |

---

## 5. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **Routing table** | Database of known routes stored in the router |
| **Default gateway** | The router your device sends traffic to when it doesn't know the destination |
| **Hop** | Each router a packet passes through on its journey |
| **TTL (Time to Live)** | A counter decremented at each hop — prevents packets looping forever |
| **Next hop** | The IP address of the next router in the path |
| **Metric** | A value used to determine the best route (lower = better) |
| **Convergence** | When all routers in a network agree on the same routing information |
| **BGP hijacking** | An attack where a malicious actor announces false routes to redirect internet traffic |
| **CIDR** | Classless Inter-Domain Routing — method for allocating IP addresses and routing |

---

## 6. 🔐 SOC Analyst Relevance

Routers are a critical point in network security. As a SOC analyst, here's what to watch for:

| Threat | What it means | Where you'd see it |
|--------|---------------|-------------------|
| **BGP Hijacking** | Attackers announce fake routes to redirect internet traffic through their infrastructure | BGP logs, threat intel feeds |
| **Route poisoning** | Malicious route updates injected into routing protocols to cause outages | Router syslogs, SIEM alerts |
| **ACL bypass** | Traffic reaching segments it shouldn't due to misconfigured rules | Firewall and router logs |
| **Unauthorized NAT changes** | Attacker modifies NAT rules to expose internal hosts | Config change alerts |
| **TTL-based attacks** | Manipulating TTL values to evade detection or cause loops | Packet captures (Wireshark) |
| **Rogue router** | Unauthorized router added to network to intercept traffic | Network discovery scans |

**Key logs to monitor as a SOC analyst:**
- Router syslog messages (interface up/down, routing changes)
- NetFlow data (traffic volume and pattern anomalies)
- BGP route change notifications
- ACL deny logs (blocked traffic attempts)

> In a SIEM like Splunk, unexpected routing table changes or a sudden spike in denied ACL entries are strong indicators of compromise worth investigating.

---

## 7. Common Interview Questions

**Q1. What is the difference between a router and a switch?**
> A switch connects devices within the same network (Layer 2, uses MAC addresses). A router connects different networks together (Layer 3, uses IP addresses) and decides the best path for packets to travel.

**Q2. What is a default gateway?**
> The default gateway is the IP address of the router that a device sends traffic to when the destination is outside its local network. For example, your laptop's default gateway is typically your home router's IP (192.168.1.1).

**Q3. What is NAT and why is it used?**
> NAT (Network Address Translation) allows multiple devices on a private network to share a single public IP address. It's used because IPv4 addresses are limited — NAT conserves public IPs while keeping private devices accessible to the internet.

**Q4. What is the difference between static and dynamic routing?**
> Static routing is manually configured by an admin — routes don't change unless edited. Dynamic routing uses protocols (OSPF, BGP) to automatically discover and update routes as the network changes. Dynamic is preferred for large networks.

**Q5. What is BGP hijacking and why should a SOC analyst care?**
> BGP hijacking is when an attacker announces false routing information to redirect internet traffic through their own systems — allowing interception or disruption. SOC analysts monitor BGP route changes and use threat intelligence feeds to detect hijacking events.

**Q6. What does TTL do?**
> TTL (Time to Live) is a value in every IP packet that decrements by 1 at each router hop. When it reaches 0, the packet is discarded. This prevents packets from looping indefinitely through the network.

---

## 8. Quick Revision Summary

```
Router operates at  →  Layer 3 (Network Layer)
Primary function    →  Forward packets between different networks
Addresses used      →  IP addresses (logical)
Key features        →  Routing table, NAT, ACLs, routing protocols
Routing protocols   →  RIP (small), OSPF (enterprise), BGP (internet)
NAT types           →  Static, Dynamic, PAT
SOC relevance       →  BGP hijacking, rogue routers, ACL bypass, NetFlow anomalies

Key ports to know:
  Telnet  → TCP 23  (insecure, should be blocked)
  SSH     → TCP 22  (secure alternative to Telnet)
  OSPF    → IP Protocol 89
  BGP     → TCP 179
```

---

*Previous topic → [OSI Model](../01-OSI-Model/notes.md)*
*Next topic → [Switches](./switches.md)*

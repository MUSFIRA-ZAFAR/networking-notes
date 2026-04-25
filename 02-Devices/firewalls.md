# 🛡️ Firewalls — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 04 | **Status:** ✅ Complete

---

## 1. Definition

A **firewall** is a network security device — hardware, software, or both — that monitors and controls incoming and outgoing network traffic based on a defined set of security rules. It sits at the boundary between a **trusted internal network** and an **untrusted external network** (such as the internet).

Its core job: **allow legitimate traffic through, block everything suspicious or unauthorized.**

> Think of a firewall as a security guard at the entrance of a building — checking every person's ID (packet) against a guest list (ruleset) before letting them in or turning them away.

---

## 2. How It Works — Step by Step

```
Step 1 — Traffic arrives at the firewall (inbound or outbound)
Step 2 — Firewall extracts packet information:
          ├── Source IP address
          ├── Destination IP address
          ├── Source port
          ├── Destination port
          └── Protocol (TCP, UDP, ICMP)
Step 3 — Firewall compares packet against its ruleset (top to bottom)
Step 4 — First matching rule is applied:
          ├── ALLOW → packet passes through
          └── DENY  → packet is dropped (silently or with rejection notice)
Step 5 — If no rule matches → default action applied
          ├── Default DENY (secure — block everything not explicitly allowed)
          └── Default ALLOW (insecure — allow everything not explicitly blocked)
Step 6 — Action is logged for monitoring and auditing
```

---

## 3. Types of Firewalls

### 3.1 Packet Filtering Firewall (Stateless)
The most basic type. Inspects each packet **individually** based on static rules.

**Checks:**
- Source/destination IP
- Source/destination port
- Protocol (TCP/UDP/ICMP)

**Limitation:** Has no memory. Cannot tell if a packet is part of a legitimate established connection or a spoofed/malicious packet. Each packet is judged in isolation.

```
Rule example:
ALLOW  TCP  any → 192.168.1.10  port 80   (allow HTTP to web server)
DENY   TCP  any → 192.168.1.10  port 22   (block SSH to web server)
DENY   any  any → any                     (block everything else)
```

### 3.2 Stateful Inspection Firewall
Tracks the **state of active connections** in a state table. Understands whether a packet is:
- Starting a new connection (SYN)
- Part of an established connection (ESTABLISHED)
- Closing a connection (FIN)

```
State table example:
Src IP        Src Port  Dst IP         Dst Port  Protocol  State
192.168.1.5   54231     93.184.216.34  443       TCP       ESTABLISHED
192.168.1.8   61002     8.8.8.8        53        UDP       NEW
```

If an inbound packet claims to be part of a connection that doesn't exist in the state table → **dropped immediately**.

Much more secure than stateless — this is what most standard firewalls use today.

### 3.3 Application Layer Firewall (Proxy Firewall)
Operates at **Layer 7 (Application Layer)**. Understands specific application protocols (HTTP, FTP, DNS, SMTP).

- Inspects actual content of traffic, not just headers
- Can block specific URLs, file types, or commands within a protocol
- Acts as a proxy — terminates the client connection and creates a new one to the server

### 3.4 Next-Generation Firewall (NGFW)
The modern enterprise standard. Combines traditional firewall with advanced features:

| Feature | Description |
|---------|-------------|
| **Deep Packet Inspection (DPI)** | Reads inside packet payload — detects malware, exploits in traffic |
| **Application awareness** | Identifies apps regardless of port (e.g., recognizes Tor even on port 443) |
| **User identity tracking** | Links traffic to specific users, not just IPs |
| **Integrated IDS/IPS** | Detects and blocks threats in real time |
| **SSL/TLS inspection** | Decrypts encrypted traffic to inspect it, then re-encrypts |
| **Threat intelligence feeds** | Blocks known malicious IPs/domains automatically |

Examples: Palo Alto Networks, Fortinet FortiGate, Cisco Firepower, Check Point.

### 3.5 Web Application Firewall (WAF)
Specifically protects **web applications** from Layer 7 attacks:
- SQL injection
- Cross-site scripting (XSS)
- CSRF attacks
- HTTP protocol violations

Sits in front of web servers — inspects HTTP/HTTPS requests and responses.

---

## 4. Firewall Deployment — Network Zones

### 4.1 DMZ — Demilitarized Zone
A **DMZ** is a network segment created by the firewall that sits between the internet and the internal network. Public-facing servers live here.

```
Internet
    │
    ▼
[Firewall]
    │
    ├──► DMZ (public-facing servers)
    │     ├── Web server
    │     ├── Email server
    │     └── DNS server
    │
    └──► Internal Network (private — employees, databases)
```

**Why DMZ matters:** If an attacker compromises a DMZ server, they still face another firewall barrier before reaching the internal network. Limits blast radius of a breach.

### 4.2 Common Firewall Topologies

| Topology | Description |
|----------|-------------|
| **Single firewall** | One firewall separates internet from internal network — simple but single point of failure |
| **Dual firewall (screened subnet)** | Two firewalls create a DMZ — more secure, defense in depth |
| **Firewall + DMZ** | Most common enterprise setup |
| **Host-based firewall** | Software firewall on individual devices (Windows Defender Firewall, iptables) |

---

## 5. Firewall Rules & ACLs

Firewall rules are processed **top to bottom** — first match wins.

```
Rule #  Action  Protocol  Source          Destination     Port
1       ALLOW   TCP       10.0.0.0/8      any             80, 443
2       ALLOW   TCP       10.0.0.0/8      any             25
3       DENY    TCP       any             10.0.1.0/24     23       ← block Telnet
4       ALLOW   ICMP      10.0.0.0/8      any             any
5       DENY    any       any             any             any      ← default deny
```

**Best practice:** Always end with an explicit **DENY ALL** rule — also called "implicit deny."

---

## 6. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **Stateless firewall** | Inspects each packet individually, no connection tracking |
| **Stateful firewall** | Tracks connection state — more intelligent filtering |
| **NGFW** | Next-Generation Firewall — DPI, app awareness, integrated IPS |
| **WAF** | Web Application Firewall — protects web apps from Layer 7 attacks |
| **DMZ** | Demilitarized Zone — buffer network between internet and internal LAN |
| **ACL** | Access Control List — ordered list of permit/deny rules |
| **Implicit deny** | Default rule — block anything not explicitly allowed |
| **DPI** | Deep Packet Inspection — inspects packet payload, not just headers |
| **Egress filtering** | Controlling outbound traffic leaving the network |
| **Ingress filtering** | Controlling inbound traffic entering the network |
| **Rule base** | The complete set of firewall rules |
| **Zones** | Logical segments (inside, outside, DMZ) that firewalls use to apply policies |

---

## 7. 🔐 SOC Analyst Relevance

Firewall logs are among the **most critical data sources** in any SOC. Here's what analysts look for:

| Event | What it indicates | Action |
|-------|------------------|--------|
| **High volume of DENY logs from one source IP** | Port scan or reconnaissance | Block IP, investigate, check threat intel |
| **Outbound connections to unknown foreign IPs** | Possible C2 (Command & Control) communication | Isolate host, full investigation |
| **Traffic on unusual ports** | Tunneling, covert channel, or misconfiguration | Investigate application using that port |
| **Firewall rule changes** | Admin action or attacker covering tracks | Verify with change management records |
| **Inbound traffic to DMZ servers on non-standard ports** | Exploitation attempt | Check IDS/IPS alerts, patch status |
| **Large outbound data transfers** | Data exfiltration | Immediate escalation |
| **Denied traffic from internal hosts** | Malware trying to phone home | Isolate host, malware analysis |
| **Firewall going offline** | Outage or deliberate attack | Priority P1 — network exposed |

**Key log fields SOC analysts read in firewall logs:**
```
Timestamp | Src IP | Src Port | Dst IP | Dst Port | Protocol | Action | Bytes
```

**SIEM queries a SOC analyst runs:**
```
index=firewall action=denied | stats count by src_ip | sort -count
index=firewall dst_port=4444 OR dst_port=1337   ← known C2 ports
index=firewall src_ip=internal action=allowed dst_ip=geo:foreign bytes>10MB
```

> **Real SOC scenario:** Firewall logs show an internal workstation making repeated outbound connections to an IP in an unusual country on port 443 — but only at 2am. This pattern (beaconing to C2) is exactly what a SOC analyst investigates by correlating firewall logs, DNS logs, and endpoint telemetry.

---

## 8. Common Interview Questions

**Q1. What is the difference between a stateless and stateful firewall?**
> A stateless firewall filters each packet individually based on static rules — it has no memory of previous packets. A stateful firewall tracks the state of active connections and can determine if a packet is part of a legitimate established session, making it far more secure and intelligent.

**Q2. What is a DMZ and why is it used?**
> A DMZ (Demilitarized Zone) is a network segment between the internet and the internal network where public-facing servers are placed. It provides a buffer — if a DMZ server is compromised, the attacker still faces a firewall before reaching the internal network.

**Q3. What is Deep Packet Inspection?**
> DPI is a firewall feature that goes beyond reading packet headers — it inspects the actual payload/content of packets. This allows detection of malware hidden in traffic, identification of applications regardless of port, and inspection of encrypted traffic (after decryption).

**Q4. What is implicit deny?**
> Implicit deny is the default rule at the end of a firewall ruleset that blocks all traffic not explicitly permitted by any rule above it. It's a security best practice — anything not specifically allowed is denied.

**Q5. What firewall logs would you check as a SOC analyst if you suspected a compromised host?**
> I'd look for outbound connections from that host to external IPs (especially on unusual ports or at unusual times), check for denied traffic that suggests reconnaissance, look for large data transfers suggesting exfiltration, and cross-reference destination IPs against threat intelligence feeds.

**Q6. What is the difference between a firewall and a WAF?**
> A traditional firewall operates at Layers 3-4 (and Layer 7 for NGFW) and protects the entire network perimeter. A WAF (Web Application Firewall) specifically protects web applications by inspecting HTTP/HTTPS traffic for application-layer attacks like SQL injection and XSS.

---

## 9. Quick Revision Summary

```
Firewall function    →  Monitor and control traffic based on rules
Types:
  Stateless          →  Packet-by-packet, no connection tracking
  Stateful           →  Tracks connection state — more secure
  NGFW               →  DPI, app awareness, IPS, SSL inspection
  WAF                →  Protects web apps from Layer 7 attacks

Key concepts:
  DMZ                →  Buffer zone for public-facing servers
  Implicit deny      →  Block all traffic not explicitly allowed
  Egress filtering   →  Control outbound traffic (often overlooked)
  DPI                →  Inspect packet payload, not just headers

SOC focus:
  Watch for          →  Port scans, C2 beaconing, data exfiltration,
                        rule changes, unusual outbound traffic
  Key log fields     →  Src IP, Dst IP, Port, Protocol, Action, Bytes
  SIEM correlation   →  Firewall + DNS + endpoint logs together
```

---

*Previous topic → [Switches](./switches.md)*
*Next topic → [IDS & IPS](./ids-ips.md)*

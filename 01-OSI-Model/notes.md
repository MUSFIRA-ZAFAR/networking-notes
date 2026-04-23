# 🔌 OSI Model — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 01 | **Status:** ✅ Complete

---

## 1. Definition

The **OSI Model (Open Systems Interconnection Model)** is a conceptual framework developed by the **International Organization for Standardization (ISO)** in 1984. It standardizes how different network systems communicate with each other by dividing the communication process into **7 distinct layers**.

Each layer has a specific job and only communicates with the layers directly above and below it. This separation of responsibilities makes troubleshooting, design, and understanding of networks significantly easier.

> Think of it as a factory assembly line — each station does one job, passes the product to the next, and the final output is a delivered message across a network.

---

## 2. How It Works — Step by Step

When you send data (e.g., loading a website), data travels **down** the OSI layers on the sender's side, gets transmitted, then travels **up** the layers on the receiver's side.

### Sending Side (Top → Bottom)
```
Application  →  data created by user action (HTTP request)
Presentation →  data encrypted & formatted
Session      →  session opened with destination server
Transport    →  data broken into segments, TCP/UDP assigned
Network      →  segments wrapped in packets, IP address added
Data Link    →  packets wrapped in frames, MAC address added
Physical     →  frames converted to bits and sent as signals
```

### Receiving Side (Bottom → Top)
```
Physical     →  signals received and converted to bits
Data Link    →  bits reassembled into frames, MAC checked
Network      →  frames unpacked into packets, IP checked
Transport    →  packets reassembled into segments, order verified
Session      →  session validated and maintained
Presentation →  data decrypted and decompressed
Application  →  data delivered to the application (browser shows page)
```

---

## 3. The 7 Layers — In Depth

---

### Layer 7 — Application
| Field | Detail |
|-------|--------|
| **Function** | Interface between user and network |
| **Protocols** | HTTP, HTTPS, FTP, SMTP, DNS, POP3, IMAP, SSH, Telnet |
| **Real device** | Browser, email client, any end-user application |
| **Data unit** | Data |

**What it really does:**
This is the layer your applications interact with directly. When you type a URL and press Enter, your browser formulates an HTTP GET request at this layer. It does NOT mean the app itself lives here — it means the network services the app uses live here.

**Real-world example:**
You open Gmail → Gmail uses SMTP (Layer 7) to send your email and IMAP to receive it.

---

### Layer 6 — Presentation
| Field | Detail |
|-------|--------|
| **Function** | Data translation, encryption, compression |
| **Protocols** | SSL, TLS, JPEG, MPEG, ASCII, Unicode |
| **Real device** | No specific device — handled by software |
| **Data unit** | Data |

**What it really does:**
Acts as the "translator" of the network. Converts data into a format the application can read. Handles encryption (making data unreadable to outsiders) and compression (reducing data size for faster transfer).

**Real-world example:**
When you visit https://google.com — the "S" in HTTPS means TLS encryption is happening at Layer 6. Your data is encrypted before being sent and decrypted when received.

---

### Layer 5 — Session
| Field | Detail |
|-------|--------|
| **Function** | Opens, manages, and closes sessions between devices |
| **Protocols** | NetBIOS, PPTP, RPC, SIP |
| **Real device** | No specific device — handled by OS |
| **Data unit** | Data |

**What it really does:**
Manages the "conversation" between two devices. It establishes when a session starts, keeps it alive during transfer, and tears it down when complete. It also handles session recovery if a connection drops mid-transfer.

**Real-world example:**
When you're on a video call — Layer 5 keeps that call session alive. If your internet drops briefly, session layer protocols try to resume the call rather than starting a new one.

---

### Layer 4 — Transport
| Field | Detail |
|-------|--------|
| **Function** | End-to-end communication, error control, flow control |
| **Protocols** | TCP, UDP |
| **Real device** | No specific device — handled by OS |
| **Data unit** | Segment |

**What it really does:**
Responsible for reliable (or fast) delivery of data. Breaks large messages into segments and reassembles them. Uses **port numbers** to direct traffic to the right application.

**TCP vs UDP:**
| | TCP | UDP |
|--|-----|-----|
| Connection | Connection-oriented (3-way handshake) | Connectionless |
| Reliability | Guaranteed delivery | No guarantee |
| Speed | Slower | Faster |
| Use case | File downloads, web browsing, email | Video streaming, VoIP, gaming |

**Real-world example:**
Downloading a file → TCP (every packet confirmed, nothing missing).
Watching a YouTube video → UDP (speed matters more than perfection, a dropped frame is acceptable).

---

### Layer 3 — Network
| Field | Detail |
|-------|--------|
| **Function** | Logical addressing and routing |
| **Protocols** | IP (IPv4, IPv6), ICMP, OSPF, BGP, RIP |
| **Real device** | **Router** |
| **Data unit** | Packet |

**What it really does:**
Responsible for finding the best path for data across multiple networks. Uses **IP addresses** (logical addresses) to identify source and destination. Routers read Layer 3 headers to decide where to forward packets.

**Real-world example:**
You send a message from Multan to a server in the US — Layer 3 routers along the way read your packet's IP address and decide the next hop, like GPS navigation for data.

---

### Layer 2 — Data Link
| Field | Detail |
|-------|--------|
| **Function** | Node-to-node delivery within the same network |
| **Protocols** | Ethernet, Wi-Fi (802.11), ARP, PPP |
| **Real device** | **Switch**, NIC (Network Interface Card) |
| **Data unit** | Frame |

**What it really does:**
Handles communication between devices on the **same local network** using **MAC addresses** (physical addresses burned into hardware). Has two sublayers: **LLC** (Logical Link Control) and **MAC** (Media Access Control).

**Real-world example:**
Your laptop sends data to your home router — Layer 2 puts your laptop's MAC address and the router's MAC address in the frame header so the data reaches the right device within your local network.

---

### Layer 1 — Physical
| Field | Detail |
|-------|--------|
| **Function** | Raw bit transmission over physical medium |
| **Protocols** | Ethernet cables, fiber optic, USB, Bluetooth, Wi-Fi signals |
| **Real device** | Cables, hubs, repeaters, NICs, access points |
| **Data unit** | Bit (0s and 1s) |

**What it really does:**
The actual, physical transmission of data as electrical signals, light pulses (fiber), or radio waves (Wi-Fi). No addressing, no logic — just raw bits moving from point A to point B.

**Real-world example:**
The ethernet cable plugged into your router — Layer 1. The fiber optic line from your ISP — Layer 1. Wi-Fi signal from your access point — Layer 1.

---

## 4. Types / Variations

The OSI Model is sometimes compared to the **TCP/IP Model** (also called the Internet Model), which condenses the 7 layers into 4:

| TCP/IP Layer | Equivalent OSI Layers |
|---|---|
| Application | Application + Presentation + Session (Layers 5, 6, 7) |
| Transport | Transport (Layer 4) |
| Internet | Network (Layer 3) |
| Network Access | Data Link + Physical (Layers 1, 2) |

> The TCP/IP model is what the internet actually runs on. OSI is the reference model used for learning and troubleshooting.

---

## 5. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **PDU** | Protocol Data Unit — the name for data at each layer (bit, frame, packet, segment, data) |
| **Encapsulation** | Process of wrapping data with headers as it moves down the layers |
| **Decapsulation** | Removing headers as data moves up the layers on the receiving end |
| **MAC Address** | Physical hardware address — used at Layer 2 |
| **IP Address** | Logical address — used at Layer 3 |
| **Port Number** | Identifies specific application/service — used at Layer 4 |
| **3-way handshake** | TCP connection setup: SYN → SYN-ACK → ACK |
| **Hop** | Each router a packet passes through on its journey |

---

## 6. 🔐 SOC Analyst Relevance

Understanding the OSI Model is **foundational** for SOC work. Here's why it matters on the job:

| Layer | SOC Use Case |
|-------|-------------|
| **Layer 7** | Analyzing HTTP/HTTPS traffic for malicious requests, detecting C2 (Command & Control) communication |
| **Layer 6** | Identifying suspicious or self-signed SSL certificates, detecting unencrypted sensitive data |
| **Layer 5** | Spotting abnormal session behavior, detecting session hijacking |
| **Layer 4** | Port scanning detection, identifying unusual port usage, TCP SYN flood (DDoS) attacks |
| **Layer 3** | IP-based threat intelligence, detecting spoofed IPs, analyzing routing anomalies |
| **Layer 2** | ARP poisoning detection, MAC address spoofing, VLAN hopping attacks |
| **Layer 1** | Physical security — unauthorized hardware taps, rogue devices on network |

> When an alert fires in a SIEM tool like Splunk or Microsoft Sentinel, a SOC analyst mentally maps it to an OSI layer to understand where the threat is happening and what to look for next.

---

## 7. Common Interview Questions

**Q1. What is the OSI Model and why is it important?**
> The OSI Model is a 7-layer framework that standardizes network communication. It's important because it gives engineers and analysts a common language to troubleshoot problems — you can isolate whether an issue is physical (Layer 1), addressing (Layer 3), or application-level (Layer 7).

**Q2. What's the difference between Layer 2 and Layer 3?**
> Layer 2 uses MAC addresses to deliver data within a local network — switches operate here. Layer 3 uses IP addresses to route data across different networks — routers operate here.

**Q3. What's the difference between TCP and UDP?**
> TCP is connection-oriented and guarantees delivery (used for web, email, file transfer). UDP is connectionless and prioritizes speed over reliability (used for video streaming, DNS, VoIP).

**Q4. How does the OSI Model help in cybersecurity?**
> Each layer introduces specific attack surfaces. Understanding the model helps SOC analysts identify where an attack is occurring — a DDoS might target Layer 3/4, while a phishing attack exploits Layer 7.

**Q5. What is encapsulation?**
> Encapsulation is the process of adding headers (and sometimes trailers) to data as it passes down each OSI layer. Each layer wraps the data with its own information before passing it to the layer below.

---

## 8. Quick Revision Summary

```
Layer 7 — Application   → User-facing protocols (HTTP, DNS, FTP)
Layer 6 — Presentation  → Encryption & formatting (SSL/TLS)
Layer 5 — Session       → Opens/manages/closes connections
Layer 4 — Transport     → TCP/UDP, ports, reliable delivery
Layer 3 — Network       → IP addresses, routing (Routers)
Layer 2 — Data Link     → MAC addresses, frames (Switches)
Layer 1 — Physical      → Cables, signals, raw bits

Mnemonic (top → bottom): Please Do Not Throw Sausage Pizza Away
Mnemonic (bottom → top): All People Seem To Need Data Processing
```

---

*Next topic → [Routers](../02-Devices/routers.md)*

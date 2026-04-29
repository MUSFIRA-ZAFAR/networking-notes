# 🔁 Proxies — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 07 | **Status:** ✅ Complete

---

## 1. Definition

A **proxy server** is an intermediary device or software that sits between a client and the destination server. Instead of the client connecting directly to the internet or a server, the request goes through the proxy — which forwards it on the client's behalf and returns the response.

**Core functions:**
- **Anonymity** — hides the real IP of client or server
- **Content filtering** — blocks or allows traffic based on policy
- **Caching** — stores frequently accessed content to speed up delivery
- **Security** — inspects and controls traffic in both directions
- **Logging** — records all traffic passing through for auditing

> Think of a proxy as a personal assistant — you tell them what you need (the request), they go get it for you, and bring it back. The other party only ever sees the assistant, never you directly.

---

## 2. How It Works — Step by Step

### Forward Proxy
```
Step 1 — Client configured to use proxy (or proxy intercepts transparently)
Step 2 — Client sends request to proxy server instead of destination
Step 3 — Proxy evaluates request against policy rules
          ├── Blocked? → Return deny page to client
          └── Allowed? → Continue
Step 4 — Proxy checks cache
          ├── Cached? → Return cached response immediately (faster)
          └── Not cached? → Continue
Step 5 — Proxy connects to destination server on client's behalf
Step 6 — Destination sees proxy's IP, not the client's real IP
Step 7 — Proxy receives response, caches it if appropriate
Step 8 — Proxy returns response to client
Step 9 — Transaction logged
```

### Reverse Proxy
```
Step 1 — Client sends request to what appears to be the web server
          (actually the reverse proxy's IP/domain)
Step 2 — Reverse proxy receives the request
Step 3 — Optionally performs SSL termination (decrypts HTTPS)
Step 4 — Applies WAF rules, rate limiting, authentication checks
Step 5 — Forwards request to appropriate backend server
          (load balancing, content-based routing)
Step 6 — Backend server processes request, sends response to proxy
Step 7 — Proxy returns response to client
Step 8 — Client never knows the real backend server IP
```

---

## 3. Types of Proxies

### 3.1 Forward Proxy
Sits between **internal clients and the internet**. The client is configured to send traffic through it (or it intercepts transparently).

```
[Internal Users] → [Forward Proxy] → [Internet]
```

**Use cases:**
- Web content filtering (block social media, adult content)
- Bandwidth optimization through caching
- Enforce acceptable use policies
- Hide internal network IPs from the internet
- Log all outbound web traffic for auditing

**Who uses it:** Enterprises, schools, ISPs, government networks.

### 3.2 Reverse Proxy
Sits between **the internet and internal servers**. Clients think they're connecting directly to the server — they don't know the proxy exists.

```
[Internet/Clients] → [Reverse Proxy] → [Backend Servers]
```

**Use cases:**
- Protect backend server IPs from exposure
- SSL/TLS termination (offload encryption from servers)
- Load balancing across backend servers
- Web Application Firewall (WAF) functionality
- Caching static content to reduce server load
- DDoS protection and rate limiting

**Who uses it:** Web applications, APIs, CDNs, enterprises.

**Examples:** NGINX, HAProxy, Cloudflare, AWS CloudFront.

### 3.3 Transparent Proxy
Intercepts traffic **without any client configuration** — the user doesn't know it exists. Often deployed at the network level (ISP or corporate gateway).

```
Client has NO proxy settings configured
Traffic automatically intercepted by transparent proxy
Client has no visibility into this
```

**Use cases:** ISP content filtering, parental controls, corporate monitoring.

**Security note:** Since clients aren't aware, this raises privacy concerns — transparent proxies used for SSL inspection require installing a trusted root certificate on client devices.

### 3.4 Anonymous Proxy
Forwards requests but **removes or replaces the client's real IP** from request headers. The destination server cannot identify the original client.

| Anonymity Level | What destination sees |
|-----------------|----------------------|
| **Transparent** | Real client IP + knows it's a proxy |
| **Anonymous** | Proxy IP + knows it's a proxy |
| **Elite/High anonymity** | Proxy IP + does NOT know it's a proxy |

### 3.5 SOCKS Proxy
Operates at a **lower level** than HTTP proxies — handles any type of traffic (HTTP, FTP, SMTP, torrents), not just web traffic. Does not interpret or modify traffic.

| Version | Features |
|---------|----------|
| **SOCKS4** | TCP only, no authentication |
| **SOCKS5** | TCP + UDP, authentication, DNS resolution through proxy |

**Use cases:** Tunneling non-HTTP traffic, bypassing firewalls, Tor network uses SOCKS5.

### 3.6 Web Proxy (HTTP/HTTPS Proxy)
Specifically handles **HTTP and HTTPS traffic**. Can interpret, filter, modify, and log web requests. Most common enterprise proxy type.

### 3.7 SSL Inspection Proxy (SSL Interception / MITM Proxy)
Decrypts HTTPS traffic, inspects content, then re-encrypts and forwards. Used for:
- Detecting malware in encrypted traffic
- DLP (Data Loss Prevention)
- Enforcing content policies on HTTPS sites

```
Client ──HTTPS──► Proxy (decrypt → inspect → re-encrypt) ──HTTPS──► Server
                        │
               Proxy uses its own cert signed by
               corporate CA (installed on all clients)
```

**Tools:** Squid, Zscaler, Blue Coat, Palo Alto SSL Inspection.

---

## 4. Proxy vs Related Concepts

| Concept | How it differs from a proxy |
|---------|---------------------------|
| **VPN** | Encrypts ALL traffic from device, operates at network layer, creates tunnel. Proxy only handles specific application traffic |
| **NAT** | Translates IP addresses at network layer — transparent to applications. Proxy operates at application layer and understands protocol content |
| **Firewall** | Controls what traffic is permitted. Proxy intermediates and inspects traffic content |
| **Load balancer** | Distributes traffic across servers. Reverse proxy can include load balancing but also adds security/caching |
| **CDN** | Globally distributed caching network. A type of reverse proxy at massive scale |

---

## 5. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **Forward proxy** | Proxy on behalf of clients — controls outbound traffic |
| **Reverse proxy** | Proxy on behalf of servers — protects backend infrastructure |
| **Transparent proxy** | Intercepts traffic without client knowledge or configuration |
| **SSL inspection** | Decrypt, inspect, re-encrypt HTTPS traffic |
| **Caching** | Storing copies of responses to serve future identical requests faster |
| **Content filtering** | Blocking or allowing traffic based on URL, category, or content |
| **PAC file** | Proxy Auto-Configuration — script that tells browsers when to use a proxy |
| **WPAD** | Web Proxy Auto-Discovery — protocol for automatically finding PAC file |
| **SOCKS5** | Protocol for proxying any TCP/UDP traffic, not just HTTP |
| **DLP** | Data Loss Prevention — detecting and blocking sensitive data from leaving |
| **Bypass list** | Domains/IPs that clients connect to directly, skipping the proxy |
| **Egress filtering** | Controlling what traffic leaves the network — proxy enforces this |

---

## 6. 🔐 SOC Analyst Relevance

Proxy logs are among the **richest data sources** in a SOC environment. They reveal exactly what internal users and systems are connecting to on the internet.

### Why Proxy Logs Matter
```
What proxy logs capture:
  ├── Timestamp
  ├── Source IP (which internal host made the request)
  ├── Username (if authenticated proxy)
  ├── Destination URL / domain / IP
  ├── HTTP method (GET, POST, CONNECT)
  ├── Response code (200, 403, 404...)
  ├── Bytes transferred (in and out)
  ├── User agent (browser/application making the request)
  └── Action taken (allowed, blocked, cached)
```

### Threats Detected Through Proxy Logs

| Threat | Indicator in proxy logs | SOC action |
|--------|------------------------|-----------|
| **Malware C2 beaconing** | Regular intervals connections to unknown domain, often odd hours | Isolate host, block domain, malware analysis |
| **Data exfiltration** | Large POST requests or high bytes-out to external IP | Investigate destination, check DLP alerts |
| **DNS tunneling** | Unusually long or encoded subdomains in requests | Block domain, isolate host |
| **Phishing callback** | User visited suspicious URL, then outbound connection to C2 | Check if credentials entered, reset passwords |
| **Policy violation** | User accessing blocked categories (P2P, anonymizers, gambling) | HR escalation, user awareness |
| **Tor / anonymizer usage** | Connections to known Tor exit nodes or proxy bypass services | Investigate intent, policy enforcement |
| **Malicious file download** | Proxy AV scan flagged download | Quarantine file, check if executed on endpoint |
| **Living-off-the-land** | PowerShell or curl making outbound web requests (not a browser) | Endpoint investigation, check process tree |

### SIEM Queries for Proxy Log Analysis
```
# Find all connections to newly registered domains (common for C2)
index=proxy | lookup newly_registered_domains dest_domain

# Detect beaconing — regular interval connections to same host
index=proxy | timechart span=1m count by dest_domain | anomalydetection

# Large data uploads (possible exfiltration)
index=proxy bytes_out > 10000000 | stats sum(bytes_out) by src_ip, dest_domain

# Non-browser user agents making web requests (malware behavior)
index=proxy NOT useragent IN ("Mozilla*", "Chrome*", "Safari*")

# Connections to known Tor exit nodes
index=proxy | lookup tor_exit_nodes dest_ip OUTPUT is_tor | where is_tor=true

# Users bypassing proxy (direct internet connections)
index=firewall dest_port IN (80,443) NOT src_ip=proxy_ip action=allowed
```

### Proxy in the SOC Kill Chain

```
Reconnaissance  →  Attacker researches target (proxy logs may capture)
Delivery        →  Phishing email with link → user clicks → proxy logs the visit
Exploitation    →  Malware downloaded → proxy logs the download
Installation    →  Malware installed, starts C2 beaconing → proxy logs the beacon
C2              →  Regular outbound connections → proxy logs the pattern
Exfiltration    →  Data sent out → proxy logs large uploads
```

> **Real SOC scenario:** SIEM alert fires on a host making connections to the same unknown domain every 5 minutes — classic beaconing. SOC analyst pulls proxy logs, confirms the pattern started after a user visited a suspicious URL. Cross-references with endpoint logs — finds a suspicious process making the connections. Isolates the host and initiates incident response.

---

## 7. Common Interview Questions

**Q1. What is the difference between a forward proxy and a reverse proxy?**
> A forward proxy sits between internal users and the internet — clients use it to access external resources, and it hides client IPs from the internet. A reverse proxy sits between the internet and internal servers — clients connect to it thinking it's the server, and it hides backend server IPs. Forward proxy protects clients; reverse proxy protects servers.

**Q2. What is a transparent proxy and how does it differ from a standard proxy?**
> A transparent proxy intercepts traffic without any client configuration — users don't know it exists. A standard (explicit) proxy requires clients to be configured with the proxy's address. Transparent proxies are common in corporate networks and ISPs for content filtering without user involvement.

**Q3. How are proxy logs useful in a SOC?**
> Proxy logs record every outbound HTTP/HTTPS request made by internal hosts — including destination URL, bytes transferred, user agent, and timestamp. This makes them invaluable for detecting malware C2 beaconing, data exfiltration, phishing callbacks, policy violations, and suspicious application behavior. They're often the first place a SOC analyst looks when investigating a potentially compromised host.

**Q4. What is SSL inspection and what are its security implications?**
> SSL inspection (or SSL interception) is when a proxy decrypts HTTPS traffic, inspects it for threats or policy violations, then re-encrypts and forwards it. It allows inspection of encrypted traffic that would otherwise be a blind spot. The implication is that the proxy acts as a man-in-the-middle — requiring a trusted corporate CA certificate installed on all client devices, and raising privacy considerations for personal traffic on corporate networks.

**Q5. What is the difference between a proxy and a VPN?**
> A proxy acts as an intermediary for specific application traffic (usually HTTP/HTTPS) at the application layer — it doesn't encrypt traffic by default. A VPN creates an encrypted tunnel for ALL network traffic from a device at the network layer. VPNs provide stronger privacy and security; proxies provide more granular control and visibility for specific traffic.

**Q6. How would you detect C2 beaconing through proxy logs?**
> I'd look for hosts making connections to the same external domain or IP at regular, predictable intervals — especially to domains with no legitimate business purpose, newly registered domains, or those flagged in threat intelligence feeds. I'd also look for unusual user agents (not standard browsers), connections at odd hours, and consistent small bytes-out with larger bytes-in pattern typical of C2 check-ins.

---

## 8. Quick Revision Summary

```
Proxy function      →  Intermediary between client and server

Types:
  Forward proxy     →  Client → Proxy → Internet (controls outbound)
  Reverse proxy     →  Internet → Proxy → Servers (protects backend)
  Transparent       →  Intercepts without client knowledge
  Anonymous         →  Hides client IP from destination
  SOCKS5            →  Any traffic type, not just HTTP
  SSL inspection    →  Decrypt, inspect, re-encrypt HTTPS

Key features:
  Caching           →  Faster content delivery
  Content filtering →  Block/allow by URL, category, content
  SSL termination   →  Offload encryption from backend servers
  Logging           →  Full visibility into outbound traffic

SOC value:
  Best for          →  Detecting C2 beaconing, exfiltration, phishing callbacks
  Key log fields    →  Src IP, dest URL, bytes, user agent, action, timestamp
  Watch for         →  Regular interval connections, large uploads,
                        non-browser agents, Tor usage, unknown domains
```

---

*Previous topic → [Load Balancers](./load-balancers.md)*
*Next topic → [TCP/IP Model](../03-TCP-IP/notes.md)*

---

## 🎉 Module 1 Complete!

Topics covered in this module:
- ✅ OSI Model
- ✅ Routers
- ✅ Switches
- ✅ Firewalls
- ✅ IDS & IPS
- ✅ Load Balancers
- ✅ Proxies

**Next module → TCP/IP, Subnetting, VLANs, DNS & DHCP, Network Protocols**

# ⚖️ Load Balancers — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 06 | **Status:** ✅ Complete

---

## 1. Definition

A **load balancer** is a network device or software solution that distributes incoming network traffic across multiple backend servers to ensure no single server is overwhelmed. It sits between clients and the server pool, acting as a reverse proxy and traffic director.

**Core goals:**
- **High availability** — if one server fails, traffic continues to others
- **Scalability** — add more servers to handle growing traffic
- **Performance** — spread load evenly for faster response times
- **Reliability** — eliminate single points of failure

> Think of a load balancer as a receptionist at a busy hospital — they direct every patient (request) to the next available doctor (server) so no one doctor is overwhelmed while others sit idle.

---

## 2. How It Works — Step by Step

```
Step 1 — Client sends request to load balancer's Virtual IP (VIP)
Step 2 — Load balancer receives the request
Step 3 — Load balancer checks health of all backend servers
Step 4 — Load balancer selects a backend server using its algorithm
Step 5 — Request is forwarded to the chosen server
Step 6 — Server processes the request and sends response
Step 7 — Response travels back through load balancer to client
          (client never sees the backend server's real IP)
Step 8 — Load balancer logs the transaction
```

---

## 3. Load Balancing Algorithms

### 3.1 Round Robin
Distributes requests **sequentially** across all servers in rotation.

```
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1  (cycle repeats)
```

| Pros | Cons |
|------|------|
| Simple and predictable | Ignores server load differences |
| Works well when servers are identical | Heavy sessions can overload one server |

### 3.2 Weighted Round Robin
Like Round Robin but servers with more capacity receive proportionally more traffic.

```
Server 1 (weight 3) → gets 3 out of every 5 requests
Server 2 (weight 2) → gets 2 out of every 5 requests
```

### 3.3 Least Connections
Routes each new request to the server with the **fewest active connections**.

```
Server 1: 45 connections
Server 2: 12 connections  ← next request goes here
Server 3: 38 connections
```

| Pros | Cons |
|------|------|
| Adapts to real server load | Slightly more complex to calculate |
| Better for long-lived sessions | Requires connection tracking |

### 3.4 Weighted Least Connections
Combines least connections with server capacity weighting — powerful servers handle more traffic proportionally.

### 3.5 IP Hash
Uses the **client's IP address** to always route them to the same backend server (session persistence / "sticky sessions").

```
Client 192.168.1.5 → hash → always goes to Server 2
Client 192.168.1.8 → hash → always goes to Server 1
```

| Pros | Cons |
|------|------|
| Session persistence without cookies | Uneven distribution if few clients |
| Stateful apps work correctly | Server failure loses all sessions for those IPs |

### 3.6 Least Response Time
Routes to the server with the **lowest average response time** and fewest connections — most intelligent algorithm.

### 3.7 Random
Selects a backend server randomly. Simple but effective at scale when servers are homogeneous.

---

## 4. Key Features

### 4.1 Health Checks
Load balancers continuously monitor backend servers:

```
Health check types:
  TCP check    → Can I open a connection to port 80?
  HTTP check   → Does GET /health return HTTP 200?
  HTTPS check  → Is SSL valid and responding?
  Custom check → Does the response body contain "OK"?

Intervals: check every 5-30 seconds
Threshold:  mark unhealthy after 2-3 consecutive failures
Recovery:   mark healthy again after 2-3 successful checks
```

If a server fails health checks → **automatically removed from pool**, traffic rerouted instantly.

### 4.2 Session Persistence (Sticky Sessions)
Ensures a client always reaches the **same backend server** during a session — critical for stateful applications.

Methods:
- **Cookie-based** — load balancer inserts a cookie identifying the server
- **IP Hash** — routes based on client IP
- **SSL Session ID** — uses TLS session ID for persistence

### 4.3 SSL Termination
Load balancer **decrypts HTTPS traffic** from clients, inspects/processes it, then forwards as HTTP to backend servers (or re-encrypts).

```
Client ──HTTPS──► Load Balancer (decrypt) ──HTTP──► Server
                        │
                  SSL cert lives here
                  (offloads crypto from servers)
```

Benefits: reduces CPU load on backend servers, centralizes certificate management.

### 4.4 Content Switching (Layer 7 Load Balancing)
Advanced load balancers can route based on **content of the request** — not just IP/port.

```
/api/*      → API server pool
/images/*   → Static content server pool
/videos/*   → Media server pool
/           → Web server pool
```

### 4.5 Connection Multiplexing
Load balancer maintains persistent connections to backend servers and reuses them for multiple client requests — reducing connection overhead.

---

## 5. Types of Load Balancers

| Type | Description |
|------|-------------|
| **Hardware load balancer** | Dedicated physical appliance — F5 BIG-IP, Citrix ADC. High performance, expensive |
| **Software load balancer** | Runs on standard hardware — HAProxy, NGINX, Apache. Flexible, cost-effective |
| **Cloud load balancer** | Managed service — AWS ELB, Azure Load Balancer, GCP Cloud Load Balancing |
| **DNS load balancer** | Distributes traffic by returning different IPs for the same domain (Global Server Load Balancing) |
| **Layer 4 load balancer** | Routes based on IP and TCP/UDP — fast, no content inspection |
| **Layer 7 load balancer** | Routes based on application content (HTTP headers, URLs, cookies) — smarter |

---

## 6. Load Balancer vs Related Concepts

| Concept | Difference |
|---------|-----------|
| **Load Balancer vs Reverse Proxy** | A reverse proxy forwards requests on behalf of clients to servers. A load balancer is a type of reverse proxy that specifically distributes traffic across multiple servers |
| **Load Balancer vs Firewall** | Firewall controls what traffic is allowed. Load balancer distributes allowed traffic across servers |
| **Load Balancer vs CDN** | CDN caches content at edge locations globally. Load balancer distributes requests across servers in one location |
| **Active-Active vs Active-Passive** | Active-Active: all servers handle traffic simultaneously. Active-Passive: standby servers only activate when primary fails |

---

## 7. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **VIP** | Virtual IP — the single IP address clients connect to (the load balancer's front-end IP) |
| **Server pool / Backend pool** | The group of servers the load balancer distributes traffic to |
| **Health check** | Regular probe to verify a backend server is alive and responding correctly |
| **Sticky session** | Routing a client to the same server for the duration of their session |
| **SSL termination** | Decrypting HTTPS at the load balancer — offloading crypto from backend servers |
| **Round robin** | Sequential distribution of requests across servers |
| **Least connections** | Route to the server with fewest active connections |
| **Failover** | Automatic switching to a healthy server when one fails |
| **High availability (HA)** | System design that minimizes downtime — load balancers are a key HA component |
| **Horizontal scaling** | Adding more servers to handle load (load balancers enable this) |
| **Layer 4 LB** | Distributes based on IP/TCP — faster, less intelligent |
| **Layer 7 LB** | Distributes based on application content — smarter, more flexible |
| **GSLB** | Global Server Load Balancing — distributes traffic across geographically distributed data centers |

---

## 8. 🔐 SOC Analyst Relevance

Load balancers are critical infrastructure — and critical targets. Here's what matters in a SOC:

### Threats Targeting Load Balancers

| Threat | What happens | Detection |
|--------|-------------|-----------|
| **DDoS attack** | Flood of requests overwhelms load balancer and/or backend servers | Traffic volume spike, backend health check failures, latency increase |
| **HTTP flood** | Layer 7 DDoS — legitimate-looking HTTP requests at massive scale | Requests per second anomaly, same source IP patterns |
| **SSL exhaustion** | Attacker initiates thousands of SSL handshakes — consumes CPU | CPU spike on load balancer, incomplete handshake count |
| **Session hijacking** | Attacker steals session cookies to bypass sticky session routing | Suspicious session cookie values, impossible geography |
| **Backend server targeting** | Attacker bypasses load balancer to hit backend servers directly | Traffic reaching backend servers from non-LB source IPs |
| **Configuration tampering** | Unauthorized changes to LB rules to redirect traffic | Config change alerts, unexpected routing behavior |

### What SOC Analysts Monitor

```
Key metrics to watch:
  ├── Requests per second (RPS) — sudden spike = possible DDoS
  ├── Active connections — sustained high count = possible flood
  ├── Backend server health — servers going down = attack or failure
  ├── Error rates (4xx, 5xx) — spike = attack or misconfiguration
  ├── Latency — increasing = overload condition
  └── SSL handshake rate — spike = SSL exhaustion attack

Key log sources:
  ├── Load balancer access logs (source IP, URL, response code, bytes)
  ├── Backend server logs (cross-reference with LB logs)
  ├── Health check logs (server up/down events)
  └── Configuration change logs (who changed what, when)
```

### SIEM Queries for Load Balancer Monitoring
```
# Detect high request rate from single IP (HTTP flood)
index=loadbalancer | stats count by src_ip | where count > 1000

# Find backends going down (possible DDoS impact)
index=loadbalancer event_type=health_check status=failed

# Detect 5xx errors spike (backend overload)
index=loadbalancer status_code>=500 | timechart count

# Unusual traffic to backend directly (bypassing LB)
index=network dst_ip=backend_pool NOT src_ip=loadbalancer_ip
```

> **Real SOC scenario:** Load balancer logs show 50,000 requests per minute from 200 different IPs all targeting the same endpoint — a distributed HTTP flood DDoS. SOC analyst correlates with upstream firewall logs, engages DDoS mitigation service, and works with network team to implement rate limiting rules on the load balancer.

---

## 9. Common Interview Questions

**Q1. What is a load balancer and why is it used?**
> A load balancer distributes incoming traffic across multiple backend servers to prevent any single server from being overwhelmed. It improves availability (if one server fails, others handle traffic), scalability (add servers to handle more users), and performance (faster response times through distribution).

**Q2. What is the difference between Layer 4 and Layer 7 load balancing?**
> Layer 4 load balancing routes traffic based on IP addresses and TCP/UDP ports — it's fast but doesn't inspect content. Layer 7 load balancing understands application-level content (HTTP headers, URLs, cookies) and can make smarter routing decisions — sending API requests to one server pool and static files to another.

**Q3. What is SSL termination and why does it matter?**
> SSL termination is when the load balancer decrypts HTTPS traffic from clients and forwards it as HTTP to backend servers. This offloads the computationally expensive SSL/TLS decryption from backend servers, centralizes certificate management, and allows the load balancer to inspect traffic content for routing decisions.

**Q4. What is a health check and why is it important?**
> A health check is a regular probe sent by the load balancer to backend servers to verify they're alive and responding correctly. If a server fails health checks, it's automatically removed from the pool — traffic reroutes to healthy servers without any manual intervention or downtime.

**Q5. How would a SOC analyst detect a DDoS attack targeting a load balancer?**
> I'd look for sudden spikes in requests per second in load balancer logs, an unusually high number of connections from multiple source IPs, increasing backend server health check failures, rising 5xx error rates, and latency increases. I'd correlate with upstream firewall logs and threat intelligence feeds to confirm the attack pattern and engage DDoS mitigation if needed.

**Q6. What is the difference between active-active and active-passive load balancing?**
> In active-active, all servers handle traffic simultaneously — maximum resource utilization and scalability. In active-passive, some servers are on standby and only activate when primary servers fail — simpler but wastes standby resources. Active-active is preferred for high-traffic environments.

---

## 10. Quick Revision Summary

```
Load balancer function  →  Distribute traffic across multiple servers
Key goals               →  High availability, scalability, performance

Algorithms:
  Round Robin           →  Sequential rotation
  Least Connections     →  Server with fewest active sessions
  IP Hash               →  Same client always hits same server
  Weighted              →  Higher capacity servers get more traffic

Key features:
  Health checks         →  Auto-remove failed servers
  Sticky sessions       →  Session persistence for stateful apps
  SSL termination       →  Decrypt at LB, offload from backends
  Layer 7 routing       →  Route by URL, headers, content

Types:
  Layer 4 LB            →  IP/port based — fast
  Layer 7 LB            →  Content-aware — smart
  Hardware / Software / Cloud

SOC focus:
  Main threat           →  DDoS (volumetric and HTTP flood)
  Watch for             →  RPS spikes, health check failures, 5xx errors
  Key logs              →  Access logs, health logs, config change logs
```

---

*Previous topic → [IDS & IPS](./ids-ips.md)*
*Next topic → [Proxies](./proxies.md)*

# 🚨 IDS & IPS — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 05 | **Status:** ✅ Complete

---

## 1. Definition

### IDS — Intrusion Detection System
An **IDS** is a security tool that **monitors** network traffic or system activity for suspicious behavior and raises alerts when a threat is detected. It does **not** block or prevent — it only observes and reports.

### IPS — Intrusion Prevention System
An **IPS** is a security tool that does everything an IDS does — but also **actively blocks** threats in real time. It sits **inline** with network traffic and can drop packets, terminate connections, or block IPs automatically without human intervention.

> **IDS = Security camera** — watches and records, alerts when something goes wrong.
> **IPS = Security guard** — watches, and physically stops threats as they happen.

---

## 2. How They Work — Step by Step

### IDS (Passive / Out-of-band)
```
Network traffic flows normally
         │
         ├──► Normal traffic reaches destination unaffected
         │
         └──► Copy of traffic sent to IDS (via port mirroring/TAP)
                    │
                    ▼
              IDS analyzes traffic
                    │
              ├── No threat detected → no action
              └── Threat detected → ALERT generated → sent to SIEM/SOC
```

### IPS (Active / Inline)
```
Network traffic flows through IPS (not around it)
         │
         ▼
    IPS inspects every packet in real time
         │
    ├── Clean traffic → forwarded to destination
    └── Malicious traffic → BLOCKED instantly
              ├── Packet dropped
              ├── Connection reset (TCP RST sent)
              └── Source IP blocked
```

---

## 3. Detection Methods

### 3.1 Signature-Based Detection
Compares traffic against a **database of known attack signatures** — specific patterns that match known exploits, malware, or attack tools.

```
Example signatures:
- SQL injection pattern: ' OR '1'='1
- Nmap scan pattern: SYN packets to sequential ports
- WannaCry ransomware network beacon pattern
- Shellcode in HTTP payload
```

| Pros | Cons |
|------|------|
| Very accurate for known threats | Cannot detect zero-day attacks |
| Low false positive rate | Requires constant signature updates |
| Fast detection | Attackers can obfuscate to evade |

### 3.2 Anomaly-Based Detection (Behavioral)
Establishes a **baseline of normal behavior** then alerts on deviations from that baseline.

```
Baseline learned:
  - Average daily traffic: 500MB
  - Typical working hours: 8am–6pm
  - Normal ports used: 80, 443, 22, 25

Anomaly detected:
  - 3am: 10GB outbound transfer to foreign IP → ALERT
  - Port 4444 connection from internal host → ALERT
  - 500 failed login attempts in 60 seconds → ALERT
```

| Pros | Cons |
|------|------|
| Can detect zero-day / unknown attacks | Higher false positive rate |
| Detects insider threats | Needs time to learn baseline |
| Adapts to new attack patterns | Legitimate changes can trigger alerts |

### 3.3 Policy-Based Detection
Alerts when traffic **violates a defined security policy** — regardless of signatures or anomalies.

```
Policy examples:
  - No traffic allowed on port 23 (Telnet) → any Telnet = alert
  - No USB devices on endpoints → USB connection = alert
  - No connections to Tor exit nodes → Tor traffic = alert
```

### 3.4 Heuristic-Based Detection
Uses **algorithms and rules** to evaluate the behavior of traffic or files to determine if they are potentially malicious — even without a known signature. Commonly used in endpoint security combined with IDS/IPS.

---

## 4. IDS vs IPS — Full Comparison

| Feature | IDS | IPS |
|---------|-----|-----|
| **Placement** | Out-of-band (passive — tap/mirror) | Inline (active — traffic flows through it) |
| **Response** | Alert only | Alert + Block |
| **Traffic impact** | None — just monitors a copy | Adds slight latency (milliseconds) |
| **False positive risk** | Low risk — only alerts, doesn't block | High risk — false positives block legitimate traffic |
| **Speed** | Can analyze without affecting traffic | Must be fast enough to inspect at line rate |
| **Best use** | Visibility and monitoring | Active threat prevention |

---

## 5. Types by Deployment

### 5.1 NIDS / NIPS — Network-based
Monitors traffic across the **entire network** at a strategic point (e.g., at the perimeter or between network segments).

```
Internet → [Firewall] → [NIPS inline] → Internal Network
                                │
                          Monitors all traffic
                          passing this segment
```

### 5.2 HIDS / HIPS — Host-based
Installed on **individual devices** (servers, workstations). Monitors system calls, file changes, log events, and local network traffic on that specific host.

```
Use cases:
  - Monitor critical server for file integrity changes
  - Detect malware running on an endpoint
  - Alert on privilege escalation attempts
```

### 5.3 Wireless IDS/IPS (WIDS/WIPS)
Monitors **wireless networks** for rogue access points, deauthentication attacks, evil twin attacks, and unauthorized wireless clients.

### 5.4 NBA — Network Behavior Analysis
Monitors network flow data (NetFlow, sFlow) to detect anomalies like DDoS attacks, port scanning, and unusual traffic patterns across the whole network.

---

## 6. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **True Positive (TP)** | Real attack correctly detected — good |
| **False Positive (FP)** | Legitimate traffic flagged as attack — generates noise |
| **True Negative (TN)** | Clean traffic correctly passed — good |
| **False Negative (FN)** | Real attack missed by the system — dangerous |
| **Alert fatigue** | SOC analysts overwhelmed by too many false positives |
| **Signature** | A pattern that uniquely identifies a known attack |
| **Baseline** | The "normal" behavior profile used for anomaly detection |
| **Port mirroring / SPAN** | Switch feature that copies traffic to IDS monitoring port |
| **Network TAP** | Hardware device that passively copies traffic to IDS |
| **Inline mode** | IPS sits directly in the traffic path |
| **Tuning** | Process of adjusting IDS/IPS rules to reduce false positives |
| **Evasion** | Techniques attackers use to bypass IDS/IPS detection |

---

## 7. 🔐 SOC Analyst Relevance

IDS/IPS alerts are the **daily bread** of SOC operations. Understanding how they work is essential for every analyst.

### Alert Triage Process
```
IDS/IPS Alert fires
        │
        ▼
Step 1 — Is this alert a known false positive? (check history)
        │
        ▼
Step 2 — Gather context:
         ├── What is the source IP? (internal/external, reputation)
         ├── What is the destination? (critical asset?)
         ├── What is the signature/rule that fired?
         └── What time did it happen? (business hours vs 3am)
        │
        ▼
Step 3 — Correlate with other logs:
         ├── Firewall logs — was traffic allowed or blocked?
         ├── DNS logs — what domain was queried?
         ├── Endpoint logs — what process made the connection?
         └── Authentication logs — any failed logins around same time?
        │
        ▼
Step 4 — Verdict:
         ├── False Positive → document, consider tuning the rule
         ├── True Positive (low severity) → remediate, document
         └── True Positive (high severity) → escalate, incident response
```

### Common IDS/IPS Alerts in a SOC

| Alert | What it means | SOC action |
|-------|--------------|-----------|
| **Port scan detected** | Attacker mapping the network | Check source IP, correlate with firewall, block if external |
| **SQL injection attempt** | Web app attack | Check WAF logs, verify if request reached DB, patch |
| **Suspicious PowerShell execution** | Possible malware or lateral movement | Isolate endpoint, full forensic investigation |
| **Beaconing to known C2 IP** | Host likely compromised | Immediate isolation, malware analysis |
| **Brute force login attempt** | Credential stuffing or password spray | Check account lockout status, source IP reputation |
| **Unusual data transfer volume** | Possible data exfiltration | Identify destination, block egress, escalate |
| **Known exploit signature matched** | Active exploitation attempt | Patch verification, check if exploit succeeded |

### Reducing Alert Fatigue — Tuning Tips
- Whitelist known-good IPs generating false positives
- Adjust thresholds for anomaly rules (e.g., raise brute force threshold)
- Disable signatures irrelevant to your environment
- Use threat intelligence to prioritize high-confidence alerts
- Correlate alerts — single events mean less than patterns

### Real-world IDS/IPS Tools
| Tool | Type | Notes |
|------|------|-------|
| **Snort** | NIDS/NIPS (open source) | Most widely used, signature-based |
| **Suricata** | NIDS/NIPS (open source) | Multi-threaded, faster than Snort |
| **Zeek (Bro)** | NIDS | Behavioral analysis, generates rich logs |
| **OSSEC** | HIDS (open source) | File integrity, log analysis |
| **Palo Alto Threat Prevention** | NGFW integrated IPS | Enterprise |
| **Cisco Firepower** | NGFW integrated IPS | Enterprise |
| **CrowdStrike Falcon** | HIPS / EDR | Cloud-native endpoint protection |

---

## 8. Common Interview Questions

**Q1. What is the difference between IDS and IPS?**
> An IDS monitors and alerts on suspicious activity but takes no action to block it — it operates out-of-band. An IPS sits inline with traffic and actively blocks threats in real time. The trade-off: IPS adds latency and false positives can block legitimate traffic, while IDS has zero impact on traffic flow but requires human or automated response after detection.

**Q2. What is the difference between signature-based and anomaly-based detection?**
> Signature-based detection matches traffic against known attack patterns — it's accurate for known threats but blind to zero-days. Anomaly-based detection learns a baseline of normal behavior and alerts on deviations — it can catch new threats but generates more false positives.

**Q3. What is a false positive and why does it matter in a SOC?**
> A false positive is when the IDS/IPS flags legitimate traffic as an attack. In a SOC, too many false positives cause alert fatigue — analysts become desensitized and may miss real threats buried in noise. Tuning IDS/IPS rules to reduce false positives is a critical ongoing task.

**Q4. What is the difference between NIDS and HIDS?**
> NIDS (Network-based IDS) monitors traffic at the network level — it sees everything passing through a network segment. HIDS (Host-based IDS) is installed on individual devices and monitors local activity — system calls, file changes, process activity. Both complement each other — NIDS gives broad visibility, HIDS gives deep endpoint insight.

**Q5. How would you handle an IDS alert for a port scan?**
> First I'd check the source IP — is it internal or external? Check its reputation against threat intel feeds. Look at firewall logs to see if traffic was allowed or blocked. Determine which ports were scanned — were they sensitive services? If external and targeting critical assets, I'd block the IP, document the event, and escalate if it's part of a broader pattern suggesting active reconnaissance.

**Q6. What is alert fatigue and how do you address it?**
> Alert fatigue occurs when SOC analysts are overwhelmed by a high volume of alerts — mostly false positives — causing them to miss real threats. It's addressed by tuning IDS/IPS rules, whitelisting known-good traffic, correlating alerts into incidents, using threat intelligence to prioritize, and automating response for low-risk, high-confidence detections (SOAR).

---

## 9. Quick Revision Summary

```
IDS  →  Detect & Alert only (passive, out-of-band)
IPS  →  Detect & Block in real time (active, inline)

Detection methods:
  Signature-based  →  Matches known attack patterns (low FP, misses zero-days)
  Anomaly-based    →  Detects deviations from baseline (catches zero-days, high FP)
  Policy-based     →  Alerts on policy violations
  Heuristic        →  Algorithm-based behavior scoring

Deployment types:
  NIDS/NIPS  →  Network-wide monitoring
  HIDS/HIPS  →  Individual host monitoring
  WIDS/WIPS  →  Wireless network monitoring

SOC key concepts:
  True Positive   →  Real attack detected ✅
  False Positive  →  Legitimate traffic flagged ⚠️ (causes alert fatigue)
  False Negative  →  Real attack missed ❌ (most dangerous)
  Tuning          →  Reducing FP rate to improve signal-to-noise ratio

Popular tools:
  Snort / Suricata  →  Open source NIDS/NIPS
  Zeek              →  Behavioral network analysis
  OSSEC             →  Open source HIDS
```

---

*Previous topic → [Firewalls](./firewalls.md)*
*Next topic → [Load Balancers](./load-balancers.md)*

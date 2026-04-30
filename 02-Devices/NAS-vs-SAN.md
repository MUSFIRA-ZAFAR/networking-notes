# 🗄️ NAS & SAN — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 08 | **Status:** ✅ Complete

---

## 1. Definition

### NAS — Network Attached Storage
A **NAS** is a dedicated file storage device connected directly to a standard Ethernet network. It provides centralized file storage accessible to multiple users and devices simultaneously using standard file-sharing protocols.

### SAN — Storage Area Network
A **SAN** is a specialized high-speed network dedicated entirely to connecting servers to storage devices. Servers access SAN storage at the **block level** — the storage appears as a locally attached disk to the server, even though it's physically remote.

> **NAS** = a smart shared drive on your network — easy, affordable, file-based.
> **SAN** = a high-speed private storage highway for servers — fast, expensive, block-based.

---

## 2. How They Work — Step by Step

### NAS
```
Step 1 — NAS device connects to the standard LAN via Ethernet
Step 2 — NAS runs its own OS (e.g., TrueNAS, Synology DSM)
Step 3 — NAS shares storage using file protocols (SMB, NFS, AFP)
Step 4 — Client devices discover NAS on network (via DNS or IP)
Step 5 — User authenticates and mounts the share
Step 6 — Files accessed as if on a local or network drive
Step 7 — NAS manages file system, permissions, and access logs
```

### SAN
```
Step 1 — SAN fabric created using Fibre Channel switches or iSCSI over Ethernet
Step 2 — Storage arrays connected to SAN fabric
Step 3 — Servers connect to SAN via HBA (Host Bus Adapter) or NIC
Step 4 — Server sends block-level read/write commands to storage
Step 5 — SAN switch routes traffic to correct storage array
Step 6 — Storage responds with raw blocks of data
Step 7 — Server's OS handles file system on top of the raw blocks
Step 8 — To the server OS — it looks like a local hard drive
```

---

## 3. NAS — Deep Dive

### 3.1 Protocols Used

| Protocol | Full name | Used for |
|----------|-----------|---------|
| **SMB** | Server Message Block | Windows file sharing (most common) |
| **NFS** | Network File System | Linux/Unix file sharing |
| **AFP** | Apple Filing Protocol | Legacy macOS sharing |
| **FTP/SFTP** | File Transfer Protocol | Remote file transfer |
| **iSCSI** | Internet SCSI | Block-level access over Ethernet (advanced NAS) |

### 3.2 NAS Use Cases
- Home/office shared storage (documents, media)
- Backup destination for workstations and servers
- Media streaming server (Plex, Jellyfin)
- Surveillance footage storage (NVR)
- Small business file server replacement
- Personal cloud storage (Nextcloud on NAS)

### 3.3 NAS Hardware Components
```
NAS device contains:
  ├── CPU (low power — ARM or Intel Atom)
  ├── RAM (for caching and OS)
  ├── Multiple drive bays (2 to 24+ drives)
  ├── RAID controller
  ├── Ethernet port(s) — 1GbE, 10GbE, or 25GbE
  └── Dedicated OS (TrueNAS, Synology DSM, QNAP QTS)
```

### 3.4 RAID on NAS

| RAID Level | Description | Min Drives | Fault Tolerance |
|------------|-------------|------------|-----------------|
| **RAID 0** | Striping — speed, no redundancy | 2 | None |
| **RAID 1** | Mirroring — full copy on each drive | 2 | 1 drive failure |
| **RAID 5** | Striping + parity — balance of speed & redundancy | 3 | 1 drive failure |
| **RAID 6** | Striping + double parity | 4 | 2 drive failures |
| **RAID 10** | Mirror of stripes — fast + redundant | 4 | Up to 50% drives |

### 3.5 Popular NAS Vendors
- **Synology** — most user-friendly, great ecosystem
- **QNAP** — feature-rich, more enterprise options
- **TrueNAS (iXsystems)** — open source, ZFS-based, enterprise grade
- **Western Digital (WD My Cloud)** — consumer grade
- **Netgear ReadyNAS** — SMB focused

---

## 4. SAN — Deep Dive

### 4.1 Protocols Used

| Protocol | Description | Speed |
|----------|-------------|-------|
| **Fibre Channel (FC)** | Dedicated FC network, purpose-built for storage | 8/16/32/128 Gbps |
| **iSCSI** | SCSI commands over standard Ethernet/IP | 1/10/25/100 Gbps |
| **FCoE** | Fibre Channel over Ethernet — converged network | 10/25 Gbps |
| **NVMe-oF** | NVMe over Fabrics — ultra-low latency | 100+ Gbps |

### 4.2 SAN Components

```
SAN architecture:
  ├── Storage Arrays — physical disks organized in arrays (EMC, NetApp, HPE)
  ├── SAN Switches (Fibre Channel switches or Ethernet switches for iSCSI)
  ├── HBA — Host Bus Adapter (in servers — like a NIC but for FC)
  ├── Zoning — logical segmentation within SAN fabric (like VLANs for FC)
  └── LUN — Logical Unit Number (virtual disk presented to a server)
```

### 4.3 SAN Use Cases
- Enterprise databases (Oracle, SQL Server — need fast block I/O)
- Virtualization storage (VMware vSAN, Hyper-V shared storage)
- High-performance computing
- Financial transaction processing
- Large-scale backup and disaster recovery
- Boot from SAN (servers boot their OS from SAN storage)

### 4.4 Popular SAN Vendors
- **Dell EMC** (PowerStore, Unity XT) — market leader
- **NetApp** — hybrid cloud storage leader
- **HPE** (3PAR, Nimble) — enterprise storage
- **Pure Storage** — all-flash SAN, high performance
- **IBM Spectrum** — enterprise mainframe environments

---

## 5. NAS vs SAN — Full Comparison

| Feature | NAS | SAN |
|---------|-----|-----|
| **Access type** | File-level (SMB, NFS) | Block-level (FC, iSCSI) |
| **Network** | Standard Ethernet LAN | Dedicated storage network |
| **Who uses it** | End users, workstations | Servers, applications |
| **Setup complexity** | Simple — plug and play | Complex — specialized knowledge |
| **Cost** | Low to medium | High (FC) to medium (iSCSI) |
| **Performance** | Good | Excellent — lowest latency |
| **Scalability** | Limited | Highly scalable |
| **File system** | Managed by NAS OS | Managed by the server OS |
| **Typical use** | File sharing, backups, media | Databases, VMs, high IOPS workloads |
| **Protocols** | SMB, NFS, AFP, FTP | FC, iSCSI, FCoE, NVMe-oF |
| **Distance** | LAN range | Can extend across WAN (iSCSI) |

---

## 6. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **Block storage** | Raw storage presented as unformatted disk — server applies file system |
| **File storage** | Pre-formatted storage with file system — accessed via file protocols |
| **LUN** | Logical Unit Number — virtual disk slice presented by SAN to a server |
| **HBA** | Host Bus Adapter — FC card in a server (like NIC but for Fibre Channel) |
| **RAID** | Redundant Array of Independent Disks — combines drives for redundancy/speed |
| **Zoning** | SAN access control — restricts which servers can see which storage (like VLANs) |
| **Fabric** | The SAN network infrastructure — FC switches and interconnects |
| **iSCSI** | SCSI block commands encapsulated in TCP/IP — SAN over standard Ethernet |
| **Fibre Channel** | High-speed storage protocol — dedicated FC network infrastructure |
| **Thin provisioning** | Allocate storage on demand rather than upfront — saves space |
| **Snapshot** | Point-in-time copy of storage — critical for backup and recovery |
| **Deduplication** | Eliminating duplicate data blocks to save storage space |
| **Tiering** | Automatically moving hot data to fast storage and cold data to slow storage |

---

## 7. 🔐 SOC Analyst Relevance

NAS and SAN devices are **prime targets** — they store the most valuable data in any organization. As a SOC analyst, storage systems are critical assets to monitor.

### Threat Landscape for Storage Systems

| Threat | How it manifests | Detection signals |
|--------|-----------------|------------------|
| **Ransomware** | Mass encryption of files on NAS — most common attack | Sudden spike in file write operations, file extensions changing, CPU spike on NAS |
| **Data exfiltration** | Large volumes of data copied off NAS to external destination | High outbound traffic from NAS IP, unusual access times, bulk file reads |
| **Unauthorized access** | Attacker gains credentials and mounts NAS share | Failed auth attempts followed by success, access from unusual IP/location |
| **Credential abuse** | Compromised user account accesses sensitive shares | User accessing shares they've never accessed before, off-hours access |
| **SAN LUN masking bypass** | Attacker reconfigures SAN zoning to access unauthorized LUNs | SAN config changes, zoning modification alerts |
| **Snapshot deletion** | Ransomware deletes snapshots before encrypting to prevent recovery | Snapshot deletion events in storage logs |
| **NAS vulnerability exploitation** | Unpatched NAS OS exploited (common with Synology, QNAP CVEs) | IDS alerts on NAS management port, unexpected outbound connections from NAS |

### Key Monitoring Points

```
NAS monitoring:
  ├── Authentication logs — failed logins, new users, privilege changes
  ├── Access logs — which user accessed which files/folders, when
  ├── File operation logs — mass reads, writes, deletes, renames
  ├── Network traffic — unexpected connections to/from NAS IP
  ├── Admin activity — config changes, new shares created, permissions modified
  └── Snapshot logs — creation and deletion of backups

SAN monitoring:
  ├── Zoning changes — unauthorized LUN access configuration
  ├── HBA authentication — unexpected hosts connecting to SAN fabric
  ├── Performance anomalies — sudden IOPS spike (ransomware encrypting)
  └── Admin access logs — who logged into SAN management console
```

### SIEM Queries for Storage Monitoring
```
# Detect ransomware — mass file rename/extension change on NAS
index=nas_logs operation=rename | stats count by src_user | where count > 500

# Unusual after-hours access to file shares
index=nas_logs hour(timestamp) < 7 OR hour(timestamp) > 20
| stats count by src_user, share_name

# Bulk file reads (possible exfiltration)
index=nas_logs operation=read | stats sum(bytes) by src_ip
| where sum(bytes) > 1073741824  ← more than 1GB

# Snapshot deletion events (ransomware pre-encryption)
index=storage_logs event_type=snapshot action=delete

# New device accessing SAN fabric
index=san_logs event_type=login NOT src_wwn IN (known_wwn_list)
```

> **Real SOC scenario:** SIEM alert fires — a NAS device shows 50,000 file rename operations in 10 minutes from a single internal IP at 2am. All renamed files now have a `.locked` extension. Classic ransomware behavior. SOC analyst immediately isolates the source host, takes the NAS offline from the network, checks if snapshots still exist for recovery, and escalates to incident response team.

### NAS Security Best Practices (SOC recommends enforcing)
- Disable default admin accounts — rename and use strong passwords
- Enable MFA on NAS management interfaces
- Restrict NAS access to specific VLANs — not accessible from guest network
- Regularly patch NAS OS — QNAP and Synology CVEs are actively exploited
- Enable immutable snapshots (WORM — Write Once Read Many)
- Monitor for SMBv1 usage — deprecated, exploited by EternalBlue/WannaCry
- Implement network segmentation — NAS should not be internet-facing

---

## 8. Common Interview Questions

**Q1. What is the difference between NAS and SAN?**
> NAS provides file-level storage over a standard Ethernet network using protocols like SMB and NFS — any networked device can access it like a shared drive. SAN provides block-level storage over a dedicated high-speed network — servers see it as a locally attached disk. NAS is simpler and cheaper; SAN is faster and more scalable for enterprise workloads.

**Q2. What is block-level vs file-level storage?**
> File-level storage (NAS) presents data as files and folders with a file system already applied — clients access files directly via protocols like SMB. Block-level storage (SAN) presents raw disk blocks — the server's OS applies its own file system on top. Block-level is faster and more flexible for applications like databases.

**Q3. What is iSCSI and how does it differ from Fibre Channel?**
> iSCSI encapsulates SCSI block storage commands inside standard TCP/IP packets — it runs on regular Ethernet infrastructure, making it more affordable and easier to deploy. Fibre Channel is a dedicated storage networking protocol with its own specialized infrastructure (FC switches, HBAs) — it offers lower latency and higher throughput but at greater cost and complexity.

**Q4. Why are NAS devices a major ransomware target?**
> NAS devices store large volumes of critical data and are accessible to many users on the network via standard file-sharing protocols — the same accessibility that makes them useful makes them easy for ransomware to reach once a host is compromised. Many NAS devices also run unpatched operating systems with known vulnerabilities, and users often connect them without proper network segmentation.

**Q5. How would you detect ransomware attacking a NAS in a SOC?**
> I'd look for a sudden mass spike in file write or rename operations from a single source IP, file extensions changing to unknown or known ransomware extensions, snapshot deletion events (ransomware tries to destroy backups first), CPU/IO spikes on the NAS, and the source host making unusual network connections. Cross-referencing with endpoint detection on the source host would confirm ransomware execution.

**Q6. What is LUN masking and zoning in a SAN?**
> Zoning is a SAN-level access control at the switch level — it defines which servers (HBAs) can communicate with which storage ports, similar to VLANs. LUN masking is a storage array-level control — it defines which servers can see and access which LUNs (virtual disks). Together they enforce the principle of least privilege in a SAN environment.

---

## 9. Quick Revision Summary

```
NAS                     →  File-level storage on standard Ethernet network
SAN                     →  Block-level storage on dedicated high-speed network

NAS protocols           →  SMB (Windows), NFS (Linux), AFP (macOS)
SAN protocols           →  Fibre Channel, iSCSI, FCoE, NVMe-oF

Key difference          →  NAS = file system on NAS device
                            SAN = file system on the server

NAS best for            →  File sharing, backups, media, small-medium business
SAN best for            →  Databases, VMs, high-performance enterprise workloads

SOC focus:
  Biggest threat        →  Ransomware (mass file encryption)
  Also watch for        →  Exfiltration, credential abuse, snapshot deletion
  Key indicator         →  Mass file rename operations in short time window
  Critical action       →  Immutable snapshots — ransomware can't delete them

RAID quick ref:
  RAID 0  →  Speed, no fault tolerance
  RAID 1  →  Mirror, 1 drive failure tolerance
  RAID 5  →  Parity, 1 drive failure tolerance (most common NAS)
  RAID 6  →  Double parity, 2 drive failures
  RAID 10 →  Mirror + stripe, best performance + redundancy
```

---

*Previous topic → [Proxies](./proxies.md)*
*Next topic → Coming soon*

# 📑 Project Proposal
## Federated Barangay Communication Network (Multi-Relay VoIP System)

### 1. Executive Summary
The Barangay Communication Network aims to provide a real-time, secure, and resilient intercom system between the Barangay Hall and its outposts, and across multiple barangays. The system uses a centralized **Master Node** in each barangay hall, **Worker Nodes** in outposts, and **Relay Servers** that interconnect barangays via the public internet.

This ensures **24/7 communication for disaster response, emergency coordination, and barangay operations.**

---

### 2. Objectives
- Establish a reliable voice communication channel between barangay hall and its outposts.  
- Extend communication across multiple barangays using interconnected relays.  
- Reduce dependency on traditional telecom (cellphone signal, landlines) during emergencies.  
- Provide a low-cost, open-source, internet-based solution using commodity hardware.  

---

### 3. System Architecture
#### 3.1 Local Topology (Inside Barangay)
**Star Topology:**
- Master Node (Barangay Hall) acts as hub.  
- Worker Nodes (Outposts) connect only to the master.  

#### 3.2 Multi-Barangay Topology (Across Internet)
**Federated Relays:**
- Each barangay runs its own Relay Server.  
- Relays interconnect via Relay-to-Relay Forwarding.  
- Audio flows: `Node → Relay A → Relay B → Node`  

#### 3.3 Flow Diagram
```
[Worker Node] --UDP--> [Master Node] --LAN--> [Relay A] --Internet--> [Relay B] --LAN--> [Master Node] --UDP--> [Worker Node]
```

---

### 4. Technical Components
#### 4.1 Hardware
- **Master Node PC:** Intel i5 / Ryzen 5, 8GB RAM, 256GB SSD, Ethernet LAN & Internet.  
- **Worker Node Devices:** Raspberry Pi / Thin Client PC, USB Headset/Speaker-Mic, 2GB RAM.  
- **Relay Server (Cloud VPS):** 1 vCPU, 1GB RAM, 25GB SSD, Public Static IP/Domain.  

#### 4.2 Software
- **Relay:** Python-based UDP server with relay-to-relay protocol.  
- **Client:** Java Swing desktop app (with audio capture/playback).  
- **Networking:** UDP (port 5000 default), optional WireGuard VPN for secure interconnect.  
- **OS:** Master/Worker: Linux/Windows, Relay: Ubuntu Server 22.04 LTS.  

---

### 5. Requirements
#### 5.1 Network
- LAN (Barangay Hall): minimum 100 Mbps switch/router.  
- Internet: Master Node: 5 Mbps up/down minimum; Relay Server: 10 Mbps symmetrical.  

#### 5.2 Security
- Token-based authentication for nodes.  
- VPN mesh (WireGuard/Tailscale) for relay servers.  
- Encrypted UDP streams (DTLS).  

#### 5.3 Scalability
- Each relay supports 50+ concurrent nodes.  
- Additional relays can be added per barangay cluster.  
- Federation model allows unlimited expansion.  

---

### 6. Implementation Plan
**Phase 1: Pilot (1 Barangay)**  
- Deploy Master Node at Barangay Hall.  
- Install 2–3 Worker Nodes in outposts.  
- Deploy 1 Relay Server (cloud VPS).  
- Test internal barangay communication.  

**Phase 2: Multi-Barangay Interconnect**  
- Deploy Relay Servers in two barangays.  
- Enable Relay-to-Relay Forwarding.  
- Conduct test calls across barangays.  

**Phase 3: Scaling**  
- Deploy in multiple barangays.  
- Central monitoring dashboard (logs, node/relay status).  

---

### 7. Budget Estimate
| Item | Qty | Unit Cost | Total |
|------|-----|-----------|-------|
| Master Node PC | 1 | ₱25,000 | ₱25,000 |
| Worker Node Devices | 5 | ₱6,000 | ₱30,000 |
| USB Headset/Mic | 5 | ₱1,000 | ₱5,000 |
| Network Switch & Router | 1 | ₱10,000 | ₱10,000 |
| Cloud VPS (Relay) | 2 | ₱500/mo | ₱12,000/year |
| Misc (cabling, UPS) | - | ₱10,000 | ₱10,000 |
| **Total (per barangay, 1 year)** |  |  | **₱92,000** |

---

### 8. Benefits
- **Resilient Communication** even if phone networks fail.  
- **Low-cost** (uses internet, no telco fees).  
- **Scalable:** add more nodes/relays as needed.  
- **Inter-Barangay Connectivity** for disaster coordination.  

---

### 9. Risks & Mitigation
| Risk | Mitigation |
|------|------------|
| Internet downtime | Redundant ISP / fallback LTE router |
| Server attack (DDoS) | VPN tunnels / firewall rules |
| Hardware failure | Deploy backup Master Node |
| Audio latency | Optimize relay routing / upgrade ISP |

---

### 10. Conclusion
This **Federated Barangay Communication Network** provides a modern, reliable, and cost-effective intercom system. By deploying multi-relay VoIP servers, barangays can achieve real-time communication within and across barangays, improving emergency response, coordination, and governance.

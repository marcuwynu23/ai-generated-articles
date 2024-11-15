
# How to Use Wireshark, Bettercap, and Netsh to Capture Network Traffic in Promiscuous Mode

Capturing all network traffic from devices on the same network can be challenging due to modern **switches** isolating traffic. However, using tools like **Wireshark** and **Bettercap**—along with enabling **promiscuous mode** on your network interface—you can analyze and capture packets beyond just the traffic directed to your machine. This article explains how to set up and use **Wireshark, Bettercap, and Netsh** to monitor traffic in promiscuous mode.

---

## What Is Promiscuous Mode?

By default, network interfaces only capture packets addressed to their MAC address or broadcast packets. **Promiscuous mode** forces the network interface to **accept and capture all packets** on the local network, regardless of the destination. This is essential for network monitoring, troubleshooting, and security assessments.

---

## Step-by-Step Guide: Using Wireshark + Bettercap + Netsh

### Step 1: Install Wireshark and Npcap

1. **Download and install Wireshark** from the official site:  
   [https://www.wireshark.org](https://www.wireshark.org).

2. During installation:
   - **Select Npcap** as the packet capture library (required for Windows).
   - Ensure Npcap is configured to allow **raw traffic capture**.

3. **Verify Installation**:
   - Open **Wireshark** and check if your network interface (Ethernet/Wi-Fi) appears in the list.

---

### Step 2: Enable Promiscuous Mode in Wireshark

1. Launch **Wireshark**.
2. Go to **Capture > Options**.
3. Select your network interface (Ethernet or Wi-Fi).
4. Check the box for **Promiscuous Mode** under the **Options** section.
5. Click **Start** to begin capturing traffic.

---

### Step 3: Enable Promiscuous Mode with `netsh` on Windows

On **Windows**, you can enable IP forwarding and configure your network interface to capture more traffic.

1. **Open Command Prompt as Administrator**.
2. Run the following command to enable IP forwarding:
   ```cmd
   netsh interface ipv4 set interface "Ethernet" forwarding=enabled
   ```

3. **Verify Promiscuous Mode** in Wireshark:
   - Start capturing traffic in Wireshark. If working correctly, you should see **broadcast, multicast, and unicast packets** from other devices on the network.

---

### Step 4: Set Up ARP Spoofing with Bettercap

To capture traffic from other devices (not just your machine), you may need to use **ARP spoofing** to force their traffic to flow through your machine.

1. **Install Bettercap** (on WSL/Kali Linux):
   ```bash
   sudo apt update && sudo apt install bettercap
   ```

2. **Run Bettercap on Your Network Interface**:
   First, identify your network interface:
   ```bash
   ip link show
   ```

   Start Bettercap on the detected interface (e.g., `eth0` or `wlan0`):
   ```bash
   sudo bettercap -iface eth0
   ```

3. **Perform ARP Spoofing** to redirect target device traffic:
   ```bash
   set arp.spoof.targets <target_device_ip>
   arp.spoof on
   net.probe on
   ```

---

### Step 5: Start Capturing Traffic with Wireshark

1. Open **Wireshark** and select the interface you used with **Bettercap**.
2. Click **Start Capture**.

You should now see:
- **Unicast packets** from the spoofed target.
- **Broadcast and multicast packets**.
- **TCP, UDP, DNS, and HTTP/HTTPS packets**.

---

### Wireshark Filters for Network Analysis

Use these filters in Wireshark to isolate relevant traffic:

- **All HTTP Traffic**:
   ```plaintext
   http
   ```

- **All HTTPS Traffic**:
   ```plaintext
   tls
   ```

- **DNS Queries**:
   ```plaintext
   dns
   ```

- **ICMP (Ping)**:
   ```plaintext
   icmp
   ```

- **Traffic from a Specific IP Address**:
   ```plaintext
   ip.addr == <target_device_ip>
   ```

---

### Step 6: Verify Traffic Redirection

1. Use **Bettercap’s logs** to verify that ARP spoofing is active.
2. In **Wireshark**, ensure you are capturing packets from other devices on the network.

---

## Troubleshooting Tips

1. **No Traffic from Other Devices?**
   - Ensure the target device is connected to the same network.
   - Try restarting the network interface using:
     ```bash
     sudo ip link set eth0 down
     sudo ip link set eth0 up
     ```

2. **Wireshark Not Capturing HTTPS Content?**
   - Use **MITMProxy** to decrypt HTTPS traffic.

3. **Promiscuous Mode Not Working on Wi-Fi?**
   - Some Wi-Fi cards don’t support promiscuous mode. Use **Ethernet** or check for updated drivers.

---

## Conclusion

By combining **Wireshark** and **Bettercap** with the right **netsh settings** on Windows, you can capture and analyze network traffic effectively in **promiscuous mode**. This setup allows you to monitor network packets from multiple devices, troubleshoot network issues, and perform security assessments. However, ensure that you have **legal permission** to capture and analyze traffic to avoid privacy violations.

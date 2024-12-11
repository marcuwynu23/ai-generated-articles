# How to Set Up and Use ISC BIND9 on Windows

BIND9 is one of the most widely used DNS (Domain Name System) servers in the world. It allows you to create a centralized DNS server that can resolve both local and public domain names. In this guide, we'll walk through the process of downloading, installing, configuring, and running **ISC BIND9** on a Windows machine using the resources provided on **GitHub**.

---

## **1. Prerequisites**

Before you begin, ensure you have the following:

- **A Windows PC** (Windows 10/11 or Server version is recommended)
- **Administrator privileges** on the system
- **An internet connection** to download the necessary files

---

## **2. Download and Extract ISC BIND9**

1. **Go to the Repository:**
   - Visit the [GitHub repository](https://github.com/HikaruDY/isc-bind9-windows).

2. **Download the Release:**
   - Navigate to the **Releases** section.
   - Download the latest `.zip` file for **ISC BIND9 for Windows**.

3. **Extract the Files:**
   - Extract the contents of the `.zip` file to a directory like:
     ```
     C:\BIND9
     ```
   - Ensure all the files (executables, libraries, configuration files, etc.) are in this directory.

4. **Verify the Installation:**
   - Open a Command Prompt as Administrator.
   - Run the following command to check if BIND9 is working properly:
     ```
     C:\BIND9\named.exe -v
     ```
     If BIND9 is installed properly, you should see the version information.

---

## **3. Configure BIND9**

### **Step 1: Create the `named.conf` File**

1. Open a text editor (like **Notepad** or **Notepad++**).
2. Create a file called **named.conf** in the `C:\BIND9` directory.
3. Add the following configuration:

```conf
options {
    directory "C:\BIND9\zones";
    allow-query { any; };
    listen-on port 53 { any; };
    allow-recursion { any; };  # Allow recursive queries
    forwarders { 8.8.8.8; 8.8.4.4; };  # Forward unresolved queries to Google DNS
    forward only;  # Ensure it only forwards unresolved queries
    dnssec-validation auto;
};

zone "example.local" IN {
    type master;
    file "example.local.db";
};

logging {
    channel default_debug {
        file "C:\BIND9\logs\bind.log" versions 3 size 10m;
        severity debug;
    };
};
```

### **Explanation**
- **options:** Controls general settings like forwarders, listening IP, and log file location.
- **zone "example.local":** Declares a custom zone (replace `example.local` with your domain name).
- **logging:** Logs BIND9's actions to `bind.log`, which is useful for debugging.

### **Step 2: Create the Zone File**

1. Create a **zones** directory in `C:\BIND9`:
   ```
   mkdir C:\BIND9\zones
   ```
2. Inside the **zones** directory, create a file called **example.local.db**.
3. Add the following example content to `example.local.db`:

```conf
$TTL 86400 ; 1 day
@ IN SOA ns1.example.local. admin.example.local. (
    2024010101 ; Serial number
    3600       ; Refresh
    1800       ; Retry
    1209600    ; Expire
    86400 )    ; Minimum TTL

@       IN NS ns1.example.local.
ns1     IN A  192.168.1.10
@       IN A  192.168.1.10
```

**Explanation:**
- The **example.local** zone file defines an internal DNS zone for **example.local**.
- Replace **192.168.1.10** with your server's IP address.
- The **Serial number** should be incremented each time the file is updated.

---

## **4. Run BIND9 as a Service**

1. **Register BIND9 as a Service**
   - Open a Command Prompt as Administrator.
   - Run the following command:
     ```
     sc create BIND9 binPath= "C:\BIND9\named.exe -c C:\BIND9\named.conf" start= auto
     ```
   - This command registers BIND9 as a service that starts automatically when Windows boots.

2. **Start the BIND9 Service**
   - Run the following command to start the service:
     ```
     net start BIND9
     ```
   - To stop the service, use:
     ```
     net stop BIND9
     ```

3. **Check for Errors**
   - Check for any errors in the log file:
     ```
     C:\BIND9\logs\bind.log
     ```

---

## **5. Set the DNS Server IP on Client Devices**

To use the BIND9 DNS server on your local network, set the DNS server IP to **192.168.1.10** (or the IP you configured) on your client devices.

### **On Windows:**
1. Go to **Control Panel** > **Network and Sharing Center**.
2. Click on your network, then click **Properties**.
3. Select **Internet Protocol Version 4 (TCP/IPv4)** and click **Properties**.
4. Set the **Preferred DNS server** to **192.168.1.10**.
5. Click **OK** to apply the changes.

### **On Linux:**
1. Edit the `/etc/resolv.conf` file.
2. Add the following line:
   ```
   nameserver 192.168.1.10
   ```

---

## **6. Test the DNS Server**

1. **Test Locally**
   - Open **Command Prompt** and run:
     ```
     nslookup example.local 192.168.1.10
     ```
   - This should return the IP address you configured (e.g., 192.168.1.10).

2. **Test from Another Device**
   - On another device in your network, change the DNS to **192.168.1.10** (as described earlier).
   - Run the same `nslookup` command and confirm that it resolves correctly.

3. **Check External DNS Resolution**
   - Make sure public domain names (like `google.com`) also resolve. If not, ensure you have configured the **forwarders** option in **named.conf**.

---

## **7. Troubleshooting Tips**

1. **Check Logs:**
   - View the log file at `C:\BIND9\logs\bind.log` to diagnose any errors.

2. **Port 53 in Use:**
   - Ensure no other services (like DNS Client or other DNS tools) are using port 53.

3. **Permission Issues:**
   - Run **Command Prompt as Administrator** for commands requiring elevated privileges.

4. **Firewall Issues:**
   - Allow incoming traffic on port **53** for DNS queries.

---

## **8. Conclusion**

You have successfully set up a centralized DNS server on Windows using **ISC BIND9**. This server can resolve custom domain names like **example.local** while also forwarding public queries to services like Google DNS (8.8.8.8). You can now manage your own internal DNS zones and handle custom domain resolutions for your network.



# How to Use Reverse SSH for Web or TCP Server Deployment with Dynamic IP

One common challenge in web or application server deployment is the need for a **static IP address** to make services accessible to the internet. If you're running an on-premise server behind a NAT or a dynamic IP address (which changes frequently), accessing this server from the internet becomes tricky. Fortunately, **reverse SSH tunneling** offers a solution to this problem, allowing you to expose your on-premise server to the public via a **VPS with a static IP**.

This article explains how reverse SSH works, how to set it up, and clarifies which server—your VPS or on-premise—uses the majority of resources in this setup.

---

## What is Reverse SSH Tunneling?

Reverse SSH tunneling is a technique where an SSH connection is initiated from the **on-premise server** to a remote **VPS** with a static IP. This allows traffic from the VPS to be forwarded to the on-premise server, effectively exposing the on-premise service (e.g., a web server) to the internet without the need for a static IP on the local network.

In a reverse SSH setup:
- **On-premise server**: Runs the actual service (web, database, etc.) and initiates the reverse SSH connection.
- **VPS**: Acts as a proxy with a static IP address that forwards incoming requests from the internet to the on-premise server.

---

## How Reverse SSH Tunneling Works for Deployment

Imagine you have a web application running on your on-premise server, but your ISP does not provide a static IP. Instead of waiting for a static IP or dealing with dynamic DNS, you can use a VPS with a static IP as an intermediary for traffic.

Here’s how to set up reverse SSH tunneling for deployment:

1. **On-Premise Server**: Runs your application (e.g., a web server on port 8080).
2. **VPS**: A publicly accessible server with a static IP that will act as the intermediary.

When a user visits your VPS, the request is forwarded to your on-premise server, which processes it and returns the response.

---

## Setting Up Reverse SSH Tunneling

### 1. **Start the Service on Your On-Premise Server**

Make sure your application (such as an HTTP server) is running on your on-premise server.

For example, start a simple Python HTTP server on port 8080:

\`\`\`bash
python3 -m http.server 8080
\`\`\`

### 2. **Create a Reverse SSH Tunnel**

Next, you will create the reverse SSH tunnel from the on-premise server to the VPS. This tunnel will allow traffic to flow from the VPS (which has a static IP) to the on-premise server (which could have a dynamic IP).

Use the following command to establish the tunnel:

\`\`\`bash
ssh -R 9000:localhost:8080 user@vps_ip
\`\`\`

- **\`-R\`**: Sets up reverse port forwarding.
- **9000**: The port on the VPS where traffic will be received.
- **8080**: The port on the on-premise server where the service is running.
- **\`user@vps_ip\`**: The SSH login to your VPS.

This command forwards port \`9000\` on the VPS to port \`8080\` on the on-premise server.

### 3. **Access the On-Premise Server via the VPS**

Once the reverse SSH tunnel is established, you can access your on-premise server via the VPS by visiting the VPS's IP and the assigned port:

\`\`\`
http://vps_ip:9000
\`\`\`

This will forward the request to the on-premise server's HTTP service running on port 8080.

---

## Resource Usage: VPS vs. On-Premise Server

In this reverse SSH setup, it’s important to understand which server is using the most resources (CPU, memory, network bandwidth):

### **1. On-Premise Server Resource Usage**
The **on-premise server** handles the actual application logic. Whether it's an HTTP server, database server, or other service, the on-premise server is responsible for processing the incoming requests forwarded from the VPS. As a result, the on-premise server uses the following resources:
- **CPU**: For processing requests and running the application.
- **Memory (RAM)**: For handling application data and sessions.
- **Disk I/O**: For reading/writing data, especially if a database is involved.
- **Network (upstream bandwidth)**: To send responses back to the VPS via the reverse SSH tunnel.

In essence, the on-premise server does the heavy lifting, as it is responsible for serving the application.

### **2. VPS Resource Usage**
The **VPS** primarily acts as a proxy, forwarding traffic from the internet to your on-premise server. It does not run the actual application, meaning its resource usage is minimal:
- **Network (bandwidth)**: Handles incoming requests and forwards them to the on-premise server via the SSH tunnel.
- **CPU**: Low usage, as it only manages the SSH connection and forwards packets.

Thus, the VPS only uses resources for managing the SSH connection and forwarding traffic, while the on-premise server consumes most of the computational resources.

---

## Benefits of Reverse SSH for Deployment

1. **Bypass Static IP Requirement**: You don't need a static IP for your on-premise server. The VPS with its static IP exposes your services to the public.
2. **Improved Security**: The connection is initiated from your on-premise server, meaning you don't need to expose your local network or open ports for incoming traffic.
3. **Flexible Setup**: The reverse SSH tunnel can be easily set up for HTTP, database, or other TCP-based services. You can forward multiple ports if needed.

---

## Potential Issues and Considerations

1. **Network Latency**: Traffic must pass through the SSH tunnel, which can introduce some latency, especially if the VPS and on-premise server are geographically far apart.
   
2. **Bandwidth Limitations**: The VPS needs to handle incoming traffic, so ensure it has enough bandwidth. Similarly, the on-premise server's outgoing bandwidth must handle responses to the VPS.

3. **SSH Tunnel Stability**: If the SSH tunnel drops, your service becomes inaccessible. You can use tools like **autossh** to automatically reconnect the tunnel if it breaks.

4. **Firewall and Security**: Make sure to configure proper firewall rules on your VPS to limit access to only the necessary ports and services. Use **SSH keys** for authentication instead of passwords for added security.

---

## Automating the Reverse SSH Tunnel

To keep the reverse SSH tunnel active even after system reboots or network disruptions, you can automate it using **autossh** or cron jobs.

### Using \`autossh\`:
\`autossh\` automatically restarts SSH tunnels if they break. Install it on your on-premise server:

\`\`\`bash
sudo apt install autossh
\`\`\`

Then, use this command to automatically maintain the SSH tunnel:

\`\`\`bash
autossh -f -N -R 9000:localhost:8080 user@vps_ip
\`\`\`

### Using Cron for Automatic Reconnection:

You can set up a cron job to recreate the tunnel on reboot or periodically:

1. Open the crontab editor:

   \`\`\`bash
   crontab -e
   \`\`\`

2. Add this line to establish the reverse SSH tunnel at system reboot:

   \`\`\`bash
   @reboot ssh -R 9000:localhost:8080 user@vps_ip
   \`\`\`

---

## Conclusion

**Reverse SSH tunneling** is a powerful technique that allows you to expose on-premise services to the internet, even if your server is behind a NAT or dynamic IP address. In this setup, the **on-premise server** handles most of the application processing, while the **VPS** acts as a proxy to forward incoming traffic. By using reverse SSH, you can deploy applications securely and flexibly, without worrying about static IP addresses or complex network configurations.

With tools like **autossh**, you can maintain the tunnel’s reliability, ensuring your services remain accessible. This solution is ideal for small businesses, home servers, and any situation where static IPs are unavailable.

--- 

By following this guide, you’ll be able to set up reverse SSH tunneling effectively, allowing you to run services on your on-premise server and make them accessible from the internet via a VPS.

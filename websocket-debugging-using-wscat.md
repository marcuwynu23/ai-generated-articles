# How to Debug and Check WebSocket Connections Using wscat

WebSockets are crucial for real-time communication in web applications. However, debugging WebSocket connectivity issues can be challenging, especially when dealing with different environments (local development, VPS, and deployed production with HTTPS). In this guide, we will walk through how to debug WebSocket connections using `wscat` and systematically test all endpoints.

## **1. Installing wscat**

`wscat` is a simple WebSocket client for testing WebSocket servers.

Install it globally using npm:

```sh
npm install -g wscat
```

Verify installation:

```sh
wscat --version
```

---

## **2. Testing WebSockets in Different Environments**

### **2.1. Local Development (Running on Your Machine)**

If your WebSocket server is running locally (e.g., on port 3000), test it with:

```sh
wscat -c "ws://localhost:3000"
```

Expected output:

```sh
Connected (press CTRL+C to quit)
>
```

If the connection fails:

- Ensure the WebSocket server is running.
- Check if it is listening on the correct port (`netstat -tulnp | grep 3000`).
- Verify the server is bound to `0.0.0.0` or `localhost`.

### **2.2. Testing WebSockets on a VPS (Local to VPS)**

Log into your VPS and test locally:

```sh
ssh user@your-vps-ip
wscat -c "ws://localhost:8844"
```

Expected output:

```sh
Connected (press CTRL+C to quit)
```

If it fails:

- Ensure the WebSocket server is running (`sudo netstat -tulnp | grep 8844`).
- Check if your firewall is blocking WebSocket traffic:
  ```sh
  sudo ufw allow 8844/tcp
  sudo ufw allow 8844/udp
  sudo ufw reload
  ```
- Verify that your server is bound to `0.0.0.0` and not just `localhost`.

### **2.3. Testing WebSockets from a Remote Machine (External Access to VPS)**

Try accessing the WebSocket server externally from your local machine:

```sh
wscat -c "ws://your-vps-ip:8844"
```

If the connection fails:

- Ensure port 8844 is open externally (`sudo ufw status`).
- Check if Nginx is proxying WebSockets correctly (see Nginx configuration below).
- Confirm that the WebSocket server is listening on the public interface (`0.0.0.0`).

### **2.4. Testing Deployed WebSockets Over HTTPS**

If your WebSocket server is behind a reverse proxy like Nginx and using HTTPS, use:

```sh
wscat -c "wss://your-domain.com/socket.io/?EIO=4&transport=websocket"
```

If this fails:

- Ensure Nginx is configured to support WebSockets:
  ```nginx
  location /socket.io/ {
      proxy_pass http://127.0.0.1:8844;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "Upgrade";
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_read_timeout 3600;
      proxy_send_timeout 3600;
  }
  ```
- Restart Nginx:
  ```sh
  sudo systemctl restart nginx
  ```
- If using **Cloudflare**, make sure **WebSockets are enabled** in the Cloudflare dashboard.

---

## **3. Additional Debugging Commands**

### **3.1. Checking WebSocket Server Status**

```sh
netstat -tulnp | grep <port>
```

Example:

```sh
netstat -tulnp | grep 8844
```

This should show that a process is listening on the specified port.

### **3.2. Testing WebSocket Connection via cURL**

```sh
curl -i -N -H "Connection: Upgrade" -H "Upgrade: websocket" "http://localhost:8844/socket.io/?EIO=4&transport=websocket"
```

Expected output:

```sh
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
```

If you get:

```sh
HTTP/1.1 400 Bad Request
Missing or invalid Sec-WebSocket-Key header
```

The WebSocket handshake might be misconfigured.

---

## **4. Conclusion**

By systematically testing WebSocket connectivity using `wscat` at different levels (local, VPS, and HTTPS deployment), you can quickly identify where the issue lies. If your WebSocket works locally but not remotely, check firewall rules, Nginx configurations, and whether the WebSocket server is correctly listening on the right interface.

If problems persist, check logs (`journalctl -u your-websocket-service`) and inspect WebSocket handshake failures in the browser’s developer console.

By following this guide, you can efficiently debug and ensure WebSockets work seamlessly across all environments. 🚀

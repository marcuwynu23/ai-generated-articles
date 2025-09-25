# 🚀 Deploying Your Web App as a Dark Web (Tor Hidden Service)

When most people hear “dark web,” they think of shady activity. But in reality, **Tor hidden services (aka onion services)** are just a technology that lets you host a website **anonymously** and make it accessible only via the Tor Browser. This can be used for privacy, whistleblowing, secure communication, or simply experimenting with decentralized networks.  

In this article, we’ll go through how to deploy your **web app** (e.g., a Node.js or Python app) as a **Tor hidden service**.

---

## 🔹 What You Need
- A server (can be your own machine, VPS, or lab server).  
- Linux (Debian/Ubuntu preferred).  
- Your web app already running (e.g., on `localhost:3000`).  
- Tor installed on the server.  
- Tor Browser (to test access).  

---

## 🔹 Step 1: Install Tor
On Debian/Ubuntu, run:

```bash
sudo apt update
sudo apt install tor -y
```

Check that it’s running:

```bash
systemctl status tor
```

---

## 🔹 Step 2: Configure Your Web App
Make sure your app is listening on **localhost** only (not exposed publicly).  
Example (Node.js Express):

```javascript
app.listen(3000, "127.0.0.1", () => {
  console.log("App running at http://127.0.0.1:3000");
});
```

This ensures your app is not directly visible on the clearnet.

---

## 🔹 Step 3: Configure Tor Hidden Service
Edit the Tor config:

```bash
sudo nano /etc/tor/torrc
```

Scroll to the bottom and add:

```
HiddenServiceDir /var/lib/tor/mywebapp/
HiddenServicePort 80 127.0.0.1:3000
```

- `HiddenServiceDir` → where Tor stores your onion hostname & keys.  
- `HiddenServicePort` → maps **port 80 of the onion site** → to your app running on `localhost:3000`.  

Save and restart Tor:

```bash
sudo systemctl restart tor
```

---

## 🔹 Step 4: Get Your Onion Address
After Tor restarts, check the hostname:

```bash
sudo cat /var/lib/tor/mywebapp/hostname
```

You’ll see something like:

```
abcd1234efgh5678.onion
```

That’s your **.onion URL** 🎉

---

## 🔹 Step 5: Access via Tor Browser
- Install [Tor Browser](https://www.torproject.org/download/).  
- Open it and visit your new onion address (e.g. `http://abcd1234efgh5678.onion`).  
- You should see your web app running!  

---

## 🔹 Step 6: (Optional) Harden Security
- Run your app behind **NGINX** for extra control.  
- Use HTTPS inside your app (not required for onion, but good for local security).  
- Limit firewall rules so the app is only accessible via localhost.  
- Regularly update Tor for patches.  

---

## 🔹 Deep Web vs Dark Web Clarified
- **Deep Web** → Anything not indexed by Google (e.g., private apps, intranet, academic databases).  
- **Dark Web** → Hidden services like `.onion` sites that require Tor.  
Your deployed onion service is part of the **dark web** subset.  

---

## ✅ Conclusion
By following these steps, you’ve taken a regular web app and turned it into a **Tor hidden service** accessible via a `.onion` domain.  

This can be useful for:  
- Privacy-conscious projects.  
- Secure communications.  
- Learning how anonymizing networks work.  
- Building resilient apps against censorship.  

> Remember: The technology itself is neutral — how you use it defines its value.

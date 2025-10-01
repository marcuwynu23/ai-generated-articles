# 🚀 Advanced Guide: Installing Docker on Windows Using WSL2 + Ubuntu  

Running Docker on Windows can be done in two main ways: using **Docker Desktop** or by installing **Docker directly inside WSL2**.  
This guide covers the **advanced method** — installing Docker **natively inside WSL2 (Ubuntu)** while controlling it from both **Linux** and **Windows PowerShell**.  

This method gives you:  
✅ Better performance (no extra VM overhead).  
✅ Full Linux-native Docker Engine.  
✅ Freedom from Docker Desktop licensing.  
✅ Flexibility to use Docker CLI from either Windows or WSL.  

---

## 1. Prerequisites  

- Windows 10/11 with **WSL2 enabled**  
- Installed **Ubuntu** distribution from the Microsoft Store  
- PowerShell with administrator rights  

Check WSL version:  
```powershell
wsl --list --verbose
```
Ensure your distro is version 2. If not, convert it:  
```powershell
wsl --set-version Ubuntu 2
```

---

## 2. Install Docker Engine Inside WSL (Ubuntu)  

Open your Ubuntu terminal in WSL and run:  

```bash
# Update packages
sudo apt update

# Install dependencies
sudo apt install -y ca-certificates curl gnupg lsb-release

# Add Docker’s official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg |   sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set up repository
echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg]   https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

Enable service:
```bash
sudo service docker start
```

(Optional) Run without `sudo`:
```bash
sudo usermod -aG docker $USER
```
Then **restart your WSL session**.  

Test inside WSL:
```bash
docker ps
```

---

## 3. Allow Windows CLI to Talk to WSL Docker  

### Option A: Quick (via socat bridge)  
Run this inside WSL:  
```bash
sudo apt install -y socat
nohup socat TCP-LISTEN:2375,fork,reuseaddr UNIX-CONNECT:/var/run/docker.sock >/dev/null 2>&1 &
```

Now your Docker daemon is exposed at `tcp://127.0.0.1:2375`.  

From PowerShell (Windows):
```powershell
docker -H tcp://127.0.0.1:2375 ps
```

---

### Option B: Permanent (systemd override)  
Instead of running socat, you can configure `dockerd` to listen directly on localhost:2375.  

Create override file:
```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo nano /etc/systemd/system/docker.service.d/override.conf
```

Paste:
```ini
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd   -H unix:///var/run/docker.sock   -H tcp://127.0.0.1:2375   --containerd=/run/containerd/containerd.sock
```

Reload & restart:
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

Check port:
```bash
ss -tuln | grep 2375
```

---

## 4. Configure Docker Context on Windows  

On Windows PowerShell:
```powershell
docker context create wsl-docker --description "Docker in WSL2" --docker "host=tcp://127.0.0.1:2375"
docker context use wsl-docker
```

Now you can simply run:
```powershell
docker ps
```

And it will connect to Docker Engine inside WSL.

---

## 5. Security Considerations  

- Always bind to **127.0.0.1**, not `0.0.0.0`, to prevent exposing Docker to your LAN.  
- For production or multi-user setups, enable **TLS certificates** if you expose TCP.  
- If you only need Windows CLI access, prefer the **socat bridge** (simpler and safer).  

---

## ✅ Summary  

By following this guide, you’ve set up:  
- A **native Docker Engine** running inside WSL2 Ubuntu.  
- The ability to manage containers from both WSL and Windows PowerShell.  
- A flexible setup without Docker Desktop.  

This is the **developer-friendly, advanced method** of running Docker on Windows — leaner, faster, and under your full control.  

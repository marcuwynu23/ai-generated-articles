# 🏗 How to Create a Self-Hosted Git Repository Hub Using Gitea

In modern software development, hosting your own Git repository hub can give you **full control** over your code, improve **security**, and eliminate **vendor lock-in**. Instead of relying solely on GitHub or GitLab, you can run your own “mini GitHub” using **Gitea** — a lightweight, free, and open-source Git service.  

This guide walks you through **setting up Gitea with Docker Compose**, configuring repositories, enabling pull requests, branch restrictions, and even CI/CD.  

---

## 🔎 Why Gitea?  

- ✅ **Free & Open-Source** (MIT License).  
- ✅ **Lightweight** → runs even on a $5 VPS or Raspberry Pi.  
- ✅ **Full GitHub-like Features**: repos, PRs, issues, branch protection, wiki.  
- ✅ **Easy Deployment** via Docker or binaries.  
- ✅ **Extensible** → integrates with CI/CD (Drone, Jenkins, or Gitea Actions).  

In short, Gitea is perfect for **teams and companies** who want GitHub-like features **without the cost**.  

---

## ⚡ Step 1: Set Up Gitea with Docker Compose  

First, create a working directory for Gitea:  

```bash
mkdir -p ~/gitea && cd ~/gitea
```

Create `docker-compose.yml`:  

```yaml
version: "3"

services:
  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    restart: always
    environment:
      - USER_UID=1000
      - USER_GID=1000
    volumes:
      - ./data:/data
    ports:
      - "3000:3000"   # Web UI
      - "222:22"      # SSH for Git
```

Start it up:  

```bash
docker compose up -d
```

Check status:  

```bash
docker ps -a
```

If successful, Gitea will be running on port **3000**.  

---

## 🌍 Step 2: Access Gitea  

Open your browser:  
```
http://localhost:3000
```

If running on a VPS, replace `localhost` with your server’s IP.  

Follow the initial setup wizard:  

- **Database** → choose SQLite (simple) or MySQL/Postgres for production.  
- **SSH Port** → set `222`.  
- **HTTP Port** → `3000`.  
- **Repository Root Path** → leave default.  
- Create an **admin account**.  

---

## 📦 Step 3: Create a Repository  

1. Log in as admin.  
2. Click **“New Repository”**.  
3. Enter a name (e.g., `myapp`).  
4. Choose **Public** or **Private**.  
5. Click **Create**.  

You’ll get clone URLs:  
- HTTPS → `http://localhost:3000/user/myapp.git`  
- SSH → `ssh://git@localhost:222/user/myapp.git`  

---

## 💻 Step 4: Use Git with Gitea  

### Clone a repo
```bash
git clone http://localhost:3000/user/myapp.git
```

### Add, commit, push
```bash
cd myapp
echo "Hello Gitea" > README.md
git add README.md
git commit -m "Initial commit"
git push origin main
```

### Create & push a branch
```bash
git checkout -b feature/login
git push -u origin feature/login
```

### Pull updates
```bash
git pull
```

---

## 🔀 Step 5: Collaborate with Pull Requests  

Gitea supports PRs just like GitHub:  

1. Push a branch (`feature/login`).  
2. Go to your repo → click **New Pull Request**.  
3. Choose base branch (`develop` or `main`).  
4. Review changes and merge.  

---

## 🔒 Step 6: Protect Branches  

Prevent accidental pushes by enabling **branch protection**:  

1. Repo → **Settings → Branches**.  
2. Add `main` or `develop`.  
3. Enable:  
   - Block direct pushes.  
   - Require PR reviews.  
   - Restrict who can push.  

Now your workflow is **PR-only**, just like GitHub.  

---

## ⚙️ Step 7: Enable CI/CD (Optional)  

### Option 1: Gitea Actions (GitHub-like Workflows)  
Recent versions of Gitea include **Actions support**. Create `.gitea/workflows/test.yml`:  

```yaml
name: Node CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install
        run: npm install
      - name: Test
        run: npm test
```

Add a **Gitea Runner** (like GitHub self-hosted runner) to execute jobs.  

### Option 2: Drone CI  
Install **Drone CI**, point it to Gitea via webhooks, and manage pipelines with `.drone.yml`.  

---

## ✅ Summary  

With just Docker Compose, you now have:  

- A **self-hosted GitHub alternative** running on your server.  
- Web UI for repos, PRs, issues, wikis.  
- Secure branch protection rules.  
- Optional CI/CD with **Gitea Actions** or **Drone**.  

Your team can now **clone, push, pull, branch, and PR** exactly like GitHub — but fully under your control.  

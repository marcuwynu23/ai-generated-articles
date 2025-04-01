# 🚀 Using Node.js with FNM on a VPS (And Verifying with GitHub Actions)

When managing a VPS (Virtual Private Server), it’s common to need different Node.js versions for various projects. Instead of installing Node.js globally or manually managing versions, you can use a lightweight tool like [`fnm`](https://github.com/Schniz/fnm) — **Fast Node Manager**.

In this guide, you'll:
- Install `fnm` on a VPS.
- Use `fnm` to install and switch Node.js versions.
- Automatically verify the setup using **GitHub Actions**.

---

## ⚙️ Step 1: Install `fnm` on Your VPS

Connect to your VPS and run the following command to install `fnm`:

```bash
curl -o- https://fnm.vercel.app/install | bash
```

This script installs `fnm` and adds it to your shell profile (e.g., `.bashrc`, `.zshrc`, etc.).

### Then restart your shell:
```bash
source ~/.bashrc # or source ~/.zshrc
```

---

## 📦 Step 2: Install Node.js Using `fnm`

Once `fnm` is installed, you can use it to install a specific version of Node.js. For example, to install **Node.js v22**:

```bash
fnm install 22
```

To activate this version in the current session:

```bash
fnm use 22
```

Make sure it's working:

```bash
node -v # Expected output: v22.14.0
npm -v  # Expected output: 10.9.2
```

---

## 🤖 Step 3: Automate Verification with GitHub Actions

To make sure everything is working remotely, you can trigger SSH commands to your VPS from GitHub using [`appleboy/ssh-action`](https://github.com/appleboy/ssh-action).

### Create a `.github/workflows/test-node-fnm.yml` file:

```yaml
name: Test SSH Connection

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: "${{ secrets.VPS_SSH_KEY }}"
          script: |
            export PATH="/root/.local/share/fnm:$PATH"
            eval "$(fnm env)"
            fnm use 22
            echo "Node version:"
            node -v
            echo "NPM version:"
            npm -v
```

### GitHub Secrets you need to add:
- `VPS_HOST`: Your VPS public IP or domain.
- `VPS_USER`: Your VPS username (e.g. `root`).
- `VPS_SSH_KEY`: Your private SSH key (used to access the VPS, **keep this safe**).

---

## ✅ Result

Every time you push to the `main` branch, GitHub Actions will:
1. SSH into your VPS.
2. Initialize `fnm`.
3. Switch to Node.js v22.
4. Output the Node and npm versions.

You’ll see these logs in your GitHub Actions run to confirm everything’s working.

---

## 🧠 Why `fnm`?

- **Blazing fast** compared to alternatives like `nvm`.
- Written in Rust, making it super lightweight and speedy.
- Easy to integrate into CI/CD and SSH workflows.

---

## 🧪 Bonus Tip: Persist `fnm` Setup Across Sessions

To make `fnm` available automatically in every SSH session, add the following to your `.bashrc` or `.zshrc`:

```bash
export PATH="/root/.local/share/fnm:$PATH"
eval "$(fnm env)"
```

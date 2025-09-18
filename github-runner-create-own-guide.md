# 🔧 How to Implement and Use Your Own GitHub Runner

GitHub Actions provides powerful automation directly in your repository, and by default, it uses GitHub-hosted runners. However, for projects with specific requirements, longer execution times, or custom environments, **self-hosted runners** are a great option.

In this guide, you'll learn how to **set up your own GitHub Actions runner** on your local machine, virtual server, or cloud instance.

---

## 📌 Why Use a Self-Hosted Runner?

Self-hosted runners give you:

- ✅ Full control over the software and hardware.
- ✅ Faster execution by avoiding queue time.
- ✅ Ability to use custom software, services, or Docker setups.
- ✅ Longer runtime limits and better performance.

---

## 🚀 Step-by-Step Guide

### 🔹 Step 1: Prepare Your Environment

You'll need:

- A machine with **Linux**, **macOS**, or **Windows** (physical or virtual).
- Admin/root access.
- GitHub account with access to the repository you want to use the runner for.

---

### 🔹 Step 2: Navigate to Your Repository Settings

1. Go to your repository on GitHub.
2. Click on `Settings` → `Actions` → `Runners`.
3. Click on the `New self-hosted runner` button.

Select the appropriate **OS** of your runner. GitHub will provide you with a command snippet to install and configure the runner.

---

### 🔹 Step 3: Create a Dedicated Runner User (Linux)

It’s best not to run the runner as `root`. Create a separate user (e.g., `ghrunner`) and give it Docker access:

```bash
# Create user
sudo adduser --disabled-password --gecos "" ghrunner
```

# Add to docker group (if you want Docker jobs)

```bash
sudo usermod -aG docker ghrunner
```

### 🔹 Step 4: Download and Configure the Runner

Run the commands provided by GitHub. Here’s a general example for Linux:

```bash
# Switch to the user for running the GitHub runner
sudo -u ghrunner -i bash

# Create a directory for the runner
mkdir github-runner && cd github-runner

# Download the latest runner package
curl -o actions-runner-linux-x64-2.327.1.tar.gz -L https://github.com/actions/runner/releases/download/v2.327.1/actions-runner-linux-x64-2.327.1.tar.gz

# Optional: Validate the hash to ensure integrity
echo "d68ac1f500b747d1271d9e52661c408d56cffd226974f68b7dc813e30b9e0575  actions-runner-linux-x64-2.327.1.tar.gz" | shasum -a 256 -c

# Extract the installer
tar xzf ./actions-runner-linux-x64-2.327.1.tar.gz

# Configure the runner
./config.sh --url https://github.com/wesdevteam --token YOUR_GENERATED_TOKEN --name runner --unattended --work _work
```

Replace:

- `YOUR_GENERATED_TOKEN` with the token provided during setup.

---

### 🔹 Step 4: Set Up the GitHub Runner as a Service

#### Install as a service for the ghrunner user

```bash
# Install as a service for the ghrunner user
sudo ./svc.sh install ghrunner

# Start the service
sudo ./svc.sh start

# Check status
sudo systemctl status actions.runner.<your-org-repo>.service

```

### 🔹 Step 5: Use in Your Workflow

Update your workflow file (e.g., `.github/workflows/main.yml`) to target your self-hosted runner:

```yaml
name: CI using self-hosted runner

on: [push]

jobs:
  build:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4
      - name: Run a script
        run: echo "Hello from self-hosted runner!"
```

You can also tag runners (e.g., `self-hosted`, `linux`, `docker`, etc.) and target them:

```yaml
runs-on: [self-hosted, linux, docker]
```

---

## 🔒 Security Considerations

- **Limit permissions**: Only allow trusted workflows or repositories to use your runner.
- **Firewall and monitoring**: Protect and monitor your runner, especially if exposed on the internet.
- **Auto cleanup**: Consider resetting or cleaning the environment between runs (use Docker or VM snapshots).

---

## 🧼 Unregister the Runner

To remove the runner:

```bash
./config.sh remove --token YOUR_TOKEN
```

To stop the service:

```bash
sudo ./svc.sh stop
sudo ./svc.sh uninstall
```

---

## 🧠 Tips and Best Practices

- Use **labels** to group runners by capabilities (e.g., `gpu`, `docker`, `android`).
- Scale horizontally by using **multiple runners** for parallel jobs.
- Keep the runner software **up to date** to ensure security and compatibility.
- If using cloud VMs, you can **auto-provision** runners with startup scripts.

---

## ✅ Conclusion

Using your own GitHub runner offers speed, flexibility, and customizability that GitHub-hosted runners can't match. Whether you need to run long jobs, use specific software, or integrate with a private network, self-hosted runners are a powerful solution.

> 🔄 Keep your runner secure, up to date, and isolated when needed.

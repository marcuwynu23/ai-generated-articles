
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

### 🔹 Step 3: Download and Configure the Runner

Run the commands provided by GitHub. Here’s a general example for Linux:

```bash
# Create a directory for the runner
mkdir actions-runner && cd actions-runner

# Download the latest runner package
curl -o actions-runner-linux-x64-2.308.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.308.0/actions-runner-linux-x64-2.308.0.tar.gz

# Extract the installer
tar xzf ./actions-runner-linux-x64-2.308.0.tar.gz

# Configure the runner
./config.sh --url https://github.com/your-username/your-repo --token YOUR_GENERATED_TOKEN
```

Replace:
- `your-username/your-repo` with your actual repository.
- `YOUR_GENERATED_TOKEN` with the token provided during setup.

---

### 🔹 Step 4: Install Dependencies

For Ubuntu/Debian:

```bash
sudo apt-get update
sudo apt-get install -y libicu-dev libkrb5-dev libssl-dev
```

For Node.js jobs, you may need to install Node:

```bash
sudo apt install nodejs npm
```

---

### 🔹 Step 5: Start the Runner

```bash
./run.sh
```

> 💡 This starts the runner in the foreground. You can also install it as a service (recommended):

```bash
sudo ./svc.sh install
sudo ./svc.sh start
```

---

## ⚙️ Use in Your Workflow

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

# 🧠 How to Add and Remove Swap Space on a Linux Server

Swap space acts as an overflow for your system's RAM, storing inactive memory pages on disk to avoid crashes or slowdowns when RAM usage is high. In this guide, you’ll learn how to **safely create swap space**, **configure it for performance**, and **remove it when no longer needed**.

---

## 📌 Why Add Swap Space?

Even if you have decent RAM, swap space helps:
- Prevent system freezes or crashes when memory is full.
- Keep inactive background apps out of RAM.
- Enable hibernation (for desktop systems).
- Support memory-hungry processes like databases, compilers, or containers.

---

## ✅ How to Add Swap Space (Swap File Method)

### Step 1: Create a Swap File

\`\`\`bash
sudo fallocate -l 2G /swapfile
\`\`\`

> Replace \`2G\` with your desired size (e.g., 1G, 4G, etc.)

### Step 2: Secure the File

\`\`\`bash
sudo chmod 600 /swapfile
\`\`\`

### Step 3: Format as Swap

\`\`\`bash
sudo mkswap /swapfile
\`\`\`

### Step 4: Enable Swap

\`\`\`bash
sudo swapon /swapfile
\`\`\`

### Step 5: Make It Permanent

Add it to your \`/etc/fstab\` so it's available on reboot:

\`\`\`bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
\`\`\`

---

## ⚙️ Optional: Tune Swappiness (Use RAM Before Swap)

\`\`\`bash
# Check current swappiness (default is 60)
cat /proc/sys/vm/swappiness

# Set it to 10 (less swap usage)
sudo sysctl vm.swappiness=10

# Make it permanent
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
\`\`\`

---

## 🔄 How to Remove Swap Space

If you no longer need the swap file, follow these steps to cleanly remove it.

### Step 1: Turn Off Swap

\`\`\`bash
sudo swapoff /swapfile
\`\`\`

### Step 2: Delete the File

\`\`\`bash
sudo rm /swapfile
\`\`\`

### Step 3: Remove from fstab

Edit \`/etc/fstab\` to remove or comment out the swap line:

\`\`\`bash
sudo nano /etc/fstab
\`\`\`

Delete this line if it exists:

\`\`\`
/swapfile none swap sw 0 0
\`\`\`

### Step 4: (Optional) Restore Default Swappiness

If you previously set \`vm.swappiness=10\`, reset to default (usually 60):

\`\`\`bash
sudo sysctl vm.swappiness=60
\`\`\`

Or remove the entry from \`/etc/sysctl.conf\`.

---

## 🧪 Verify Your Changes

\`\`\`bash
free -h
\`\`\`

- After adding swap, you should see it listed.
- After removal, swap should show \`0B\`.

---

## 🚀 Final Thoughts

Adding swap space is a **quick and effective way to boost stability**, especially on low-memory servers. It's not a replacement for RAM, but it's a critical tool to avoid downtime, especially in production environments.

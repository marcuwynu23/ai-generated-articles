# 🛠 How to Track Down Excessive Storage Consumption on a Ubuntu linux

Running out of disk space on your VPS can break critical services like **nginx**, **databases**, or **PM2 apps**. Knowing how to track and fix excessive storage usage is essential for smooth server management.

This guide will walk you through practical steps to identify what’s eating your storage and how to clean it safely.

---

## 1. Check Overall Disk Usage

First, see which partitions are filling up:

```bash
df -h
```

Example output:

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        80G   75G  2.5G  97% /
```

👉 Usually `/` is the one you need to investigate.

---

## 2. Identify Large Directories

Check which top-level directories are consuming space:

```bash
sudo du -sh /* 2>/dev/null | sort -h
```

Common suspects include:

- `/var` → logs, databases, caches
- `/home` → user files, uploads
- `/usr` → packages and libraries

---

## 3. Drill Down Into Culprits

If `/var` is large, investigate further:

```bash
sudo du -sh /var/* 2>/dev/null | sort -h
```

Keep drilling down (e.g., `/var/log`, `/var/lib`, `/var/www`) until you find the big offenders.

---

## 4. Find Oversized Files

Locate huge files directly:

```bash
sudo find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | sort -k5 -h
```

Look for:

- Giant log files (`.log`, `.json`)
- Old backups (`.sql`, `.tar.gz`)
- Cached packages or builds

---

## 5. Inspect Logs

Logs often grow uncontrollably.

Check log sizes:

```bash
sudo du -sh /var/log/* | sort -h
```

Systemd journal logs:

```bash
sudo journalctl --disk-usage
```

Example issues:

- `/var/log/suricata/eve.json` → IDS logs bloating to tens of GB
- `/var/log/syslog` or `/var/log/auth.log` → unrotated system logs

---

## 6. Check Package Caches

On Debian/Ubuntu:

```bash
sudo du -sh /var/cache/apt
sudo apt-get clean && sudo apt-get autoremove -y
```

On CentOS/RHEL:

```bash
sudo du -sh /var/cache/yum
sudo yum clean all
```

---

## 7. Check Docker (if used)

Docker images, volumes, and containers can consume huge space:

```bash
docker system df
docker system prune -af --volumes
```

---

## 8. Use Interactive Tools

Install **ncdu** for an interactive view:

```bash
sudo apt-get install ncdu -y   # Debian/Ubuntu
sudo yum install ncdu -y       # CentOS/RHEL

sudo ncdu /
```

Navigate with arrow keys to find what’s using the most space.

---

## 9. Prevent Future Problems

- **Enable log rotation** (`logrotate` or `pm2-logrotate`)
- **Configure app logging** (reduce verbose logs like Suricata’s `eve.json`)
- **Move backups** off the VPS (e.g., S3, external storage)
- **Monitor disk usage** with tools like `monit`, `netdata`, or simple cron jobs emailing `df -h`

---

## ✅ Summary

When your VPS runs out of space:

1. Use `df -h` to see full partitions
2. Use `du -sh` to find large directories
3. Drill down into logs, caches, and user files
4. Identify large files with `find`
5. Clean safely — truncate logs, clear caches, rotate backups
6. Prevent it from happening again with log rotation and monitoring

# Comprehensive Guide to Cyberattacks on a VPS

## 1. Introduction

A Virtual Private Server (VPS) is vulnerable to various cyberattacks, including unauthorized access, denial-of-service (DoS), malware infections, and data breaches. This guide provides steps to detect, analyze, and document security incidents on a VPS.

## 2. Common Types of Cyberattacks

### 2.1 Brute-Force Attacks

Attackers try multiple username-password combinations to gain unauthorized access.

#### Detection:

- Check failed login attempts:
  ```sh
  grep "Failed password" /var/log/auth.log | awk '{print $9}' | sort | uniq -c | sort -nr
  ```
- Use Fail2Ban to block repeated attempts:
  ```sh
  sudo fail2ban-client status sshd
  ```

#### Prevention:

- Disable root login (`PermitRootLogin no` in `/etc/ssh/sshd_config`)
- Use key-based authentication instead of passwords
- Change the SSH port from `22` to a random number

### 2.2 Denial-of-Service (DoS) / Distributed DoS (DDoS)

Overwhelms the server with excessive traffic, causing service unavailability.

#### Detection:

- Monitor unusual traffic spikes:
  ```sh
  netstat -ant | awk '{print $6}' | sort | uniq -c | sort -nr
  ```
- Identify large number of connections from the same IP:
  ```sh
  netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr | head -10
  ```

#### Prevention:

- Use a firewall (UFW or iptables) to limit connections per IP:
  ```sh
  sudo ufw limit ssh/tcp
  ```
- Install DDoS mitigation tools (Fail2Ban, Cloudflare, etc.)

### 2.3 Malware and Rootkits

Attackers install malicious software to gain persistence or exfiltrate data.

#### Detection:

- Scan for rootkits:
  ```sh
  sudo rkhunter --check
  ```
- Check for unexpected processes:
  ```sh
  ps aux --sort=-%cpu | head -10
  ```

#### Prevention:

- Regularly update packages:
  ```sh
  sudo apt update && sudo apt upgrade -y
  ```
- Install an anti-malware scanner like ClamAV:
  ```sh
  sudo apt install clamav -y
  sudo clamscan -r /
  ```

### 2.4 Unauthorized File Modifications (Defacement / Backdoors)

Hackers modify files to deface the website or create hidden backdoors.

#### Detection:

- Check for recently modified files:
  ```sh
  find /var/www -type f -mtime -1
  ```
- Monitor system changes:
  ```sh
  sudo auditctl -w /etc/passwd -p wa -k password_changes
  ```

#### Prevention:

- Set file integrity monitoring with AIDE:
  ```sh
  sudo apt install aide -y
  sudo aideinit
  sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db
  ```
- Restrict file permissions (`chmod` and `chown` as needed)

## 3. Analyzing and Responding to an Attack

### 3.1 Identifying Suspicious IPs

- List all connected IPs:
  ```sh
  netstat -antp
  ```
- Block an attacker’s IP:
  ```sh
  sudo ufw deny from <IP>
  ```

### 3.2 Killing Malicious Processes

- Find processes by network usage:
  ```sh
  sudo lsof -i -n -P
  ```
- Kill a suspicious process:
  ```sh
  sudo kill -9 <PID>
  ```

### 3.3 Checking System Logs for Suspicious Activity

- View all authentication logs:
  ```sh
  sudo cat /var/log/auth.log | grep "authentication"
  ```
- Check system-wide logs:
  ```sh
  sudo cat /var/log/syslog
  ```

## 4. Documenting Security Incidents

### 4.1 Logging Attack Information

For proper documentation and future mitigation:

- Date and time of the attack
- Affected services
- IP addresses involved
- Log excerpts
- Actions taken
- Remediation steps

### 4.2 Creating an Incident Report

Example template:

```
## Incident Report
- **Date:** [YYYY-MM-DD]
- **Time:** [HH:MM UTC]
- **Affected Server:** [Hostname/IP]
- **Attack Type:** [Brute-force, DDoS, Malware, etc.]
- **Description:** [Summary of the attack]
- **Evidence:**
  - Log Entries: [Relevant log entries]
  - Affected Files/Processes: [List of changes]
- **Actions Taken:**
  - [Blocked IPs, restarted services, etc.]
- **Future Preventive Measures:**
  - [Firewall rules, authentication changes, etc.]
```

## 5. Conclusion

By regularly monitoring logs, applying security measures, and documenting incidents, you can minimize cyber threats against your VPS. Implement proactive security controls to ensure long-term protection.

Would you like an automated script to check for attacks? 🚀

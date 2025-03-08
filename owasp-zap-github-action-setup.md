# Automating OWASP ZAP Security Scans in GitHub Actions

## Introduction

Integrating **OWASP ZAP** into **GitHub Actions** allows automated security scanning for vulnerabilities in web applications. This guide explains how to use OWASP ZAP in GitHub Actions, generate security reports, and automatically create security issues.

---

## ✅ Step 1: Basic OWASP ZAP GitHub Action Setup

The following GitHub Actions workflow runs an **OWASP ZAP baseline scan** whenever code is pushed to the `security` branch.

### **1️⃣ Create `.github/workflows/security_scan.yml`**

```yaml
name: OWASP ZAP Security Scan
on:
  push:
    branches:
      - security

permissions:
  contents: read
  issues: write # ✅ Required for creating issues

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - name: OWASP ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.14.0
        with:
          target: "https://bccsa-api.cloudmateria.com"
          fail_action: false # Prevents job failure due to warnings
          issue_title: "OWASP ZAP Security Report"
          allow_issue_writing: true # ✅ Enables automatic issue creation
```

✔ **How it works:**

- Triggers **on push** to the `security` branch.
- Runs an **OWASP ZAP Baseline Scan**.
- **Creates a GitHub issue** with security findings.
- **Does not fail the workflow** on warnings.

---

## ✅ Step 2: Advanced OWASP ZAP Scans in GitHub Actions

For **more advanced security testing**, use an **Active Scan** or **Full Scan**.

### **1️⃣ OWASP ZAP Active Scan (More Aggressive)**

Use the **Active Scan** for deeper security checks, but ensure this is done in a **staging** or **test environment** (not production!).

```yaml
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - name: OWASP ZAP Active Scan
        uses: zaproxy/action-full-scan@v0.14.0
        with:
          target: "https://staging.bccsa-api.cloudmateria.com"
          fail_action: true # ✅ Fails the workflow if high risks are found
          issue_title: "⚠️ OWASP ZAP Security Issues Detected"
          allow_issue_writing: true
```

✔ **How it works:**

- Performs an **Active Scan** (detects vulnerabilities like SQLi & XSS).
- Runs against the **staging environment**.
- **Fails the job** if high-risk vulnerabilities are found.

---

## ✅ Step 3: Downloading Security Reports in GitHub Actions

OWASP ZAP allows **saving reports** as HTML, XML, or JSON.

### **1️⃣ Modify the Workflow to Download Reports**

```yaml
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - name: OWASP ZAP Baseline Scan with Reports
        uses: zaproxy/action-baseline@v0.14.0
        with:
          target: "https://bccsa-api.cloudmateria.com"
          fail_action: false
          issue_title: "OWASP ZAP Security Report"
          allow_issue_writing: true
          format: html # ✅ Generates an HTML report
          report_name: zap_report.html # ✅ Save as zap_report.html

      - name: Upload Security Report
        uses: actions/upload-artifact@v4
        with:
          name: OWASP-ZAP-Report
          path: zap_report.html
```

✔ **How it works:**

- **Saves the ZAP report** as an HTML file (`zap_report.html`).
- **Uploads the report** as an artifact in GitHub Actions.
- You can **download the report** from the GitHub Actions page.

---

## ✅ Step 4: Running ZAP in a Docker Container (Self-Hosted Runners)

If you use **self-hosted runners**, deploy **OWASP ZAP as a Docker container** for faster execution.

```yaml
jobs:
  security:
    runs-on: self-hosted
    steps:
      - name: Start OWASP ZAP in Docker
        run: |
          docker run -d -p 8080:8080 zaproxy/zap-stable zap.sh -daemon -host 0.0.0.0 -port 8080

      - name: Run OWASP ZAP Scan
        run: |
          curl "http://localhost:8080/JSON/ascan/action/scan/?url=https://staging.bccsa-api.cloudmateria.com"

      - name: Stop OWASP ZAP
        run: |
          docker stop $(docker ps -q --filter ancestor=zaproxy/zap-stable)
```

✔ **How it works:**

- **Runs ZAP inside Docker** on a **self-hosted runner**.
- **Scans the target application.**
- **Stops the container** when done.

---

## 🚀 Final Summary

### **Basic Workflow**

✅ Runs an **automated OWASP ZAP scan** on code pushes.  
✅ **Creates GitHub Issues** if security vulnerabilities are detected.

### **Advanced Features**

✅ **Active Scan** for deeper vulnerability testing.  
✅ **Download reports** in HTML format.  
✅ **Run OWASP ZAP in Docker** on self-hosted runners.

With this setup, **OWASP ZAP continuously scans your application for security risks** in GitHub Actions, ensuring you catch vulnerabilities early. 🔐🚀

---

### 🔥 Next Steps:

- **Automate weekly security scans** using GitHub scheduled actions (`on: schedule`)
- **Integrate with Slack/Teams** to send security alerts
- **Add API authentication** to protect OWASP ZAP usage (`-config api.key=YOUR_API_KEY`)

Need help? Let me know! 🚀

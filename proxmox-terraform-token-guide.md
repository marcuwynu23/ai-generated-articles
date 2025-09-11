# 🔑 Using Proxmox API Tokens with Terraform

This guide explains how to create and use **Proxmox API tokens** in place of username/password for Terraform and automation scripts.

---

## 1️⃣ Create a Terraform User (once)

1. Log in to **Proxmox Web UI** (`https://<proxmox-ip>:8006`).
2. Go to: **Datacenter → Permissions → Users → Add**.
   - **User ID**: `terraform`
   - **Realm**: `pve`
   - **Expire**: leave blank (never expire)
   - Password: required but not used once you switch to tokens
3. Click **Add**.

---

## 2️⃣ Assign Permissions

1. Go to: **Datacenter → Permissions → Add**.
2. Path: `/` (or restrict to `/vms` if you want VM-only management).
3. User: `terraform@pve`.
4. Role: 
   - `Administrator` → full control (labs, staging).
   - `PVEVMAdmin` → VM lifecycle only (recommended for production).
5. Save.

---

## 3️⃣ Create an API Token

1. Go to: **Datacenter → Permissions → API Tokens → Add**.
2. Fill in:
   - **User**: `terraform@pve`
   - **Token ID**: `tf` (only the short name, not the full string!)
   - Check **Privilege Separation** ✅
3. Click **Add**.
4. Copy the **Secret** (only shown once).
5. The full Token string is now:

```
terraform@pve!tf
```

---

## 4️⃣ Test the Token via `curl`

```sh
curl -k -H "Authorization: PVEAPIToken=terraform@pve!tf=<YOUR_SECRET>" \
  https://<proxmox-ip>:8006/api2/json/version
```

Expected JSON:
```json
{"data":{"release":"8.1","repoid":"...","version":"8.1-2"}}
```

---

## 5️⃣ Use in Terraform

### `variables.tf`
```hcl
variable "pm_api_url" {
  default     = "https://192.168.1.80:8006/api2/json"
  description = "Proxmox API URL"
}

variable "pm_api_token_id" {
  description = "Proxmox API token ID (e.g. terraform@pve!tf)"
  type        = string
}

variable "pm_api_token_secret" {
  description = "Proxmox API token secret"
  type        = string
  sensitive   = true
}

variable "pm_tls_insecure" {
  description = "Disable TLS verification (true if self-signed certs)"
  type        = bool
  default     = true
}
```

### `terraform.tfvars`
```hcl
pm_api_token_id     = "terraform@pve!tf"
pm_api_token_secret = "YOUR_SECRET"
```

### `main.tf`
```hcl
provider "proxmox" {
  pm_api_url          = var.pm_api_url
  pm_api_token_id     = var.pm_api_token_id
  pm_api_token_secret = var.pm_api_token_secret
  pm_tls_insecure     = var.pm_tls_insecure
}
```

---

## 6️⃣ Use in Scripts (Health Check)

Instead of `root@pam` + password:

```sh
#!/bin/sh
API_URL="https://192.168.1.80:8006/api2/json"
TOKEN_ID="terraform@pve!tf"
TOKEN_SECRET="YOUR_SECRET"

RESPONSE=$(curl -s -k -H "Authorization: PVEAPIToken=${TOKEN_ID}=${TOKEN_SECRET}" \
  "$API_URL/version")

if echo "$RESPONSE" | grep "version" >/dev/null 2>&1; then
  echo '{"status":"online"}'
else
  echo '{"status":"unreachable"}'
fi
```

---

## 🔒 Best Practices
- Store token secret in **environment variables**:
  ```sh
  export TF_VAR_pm_api_token_secret="YOUR_SECRET"
  ```
- Add `terraform.tfvars` to `.gitignore`.
- Grant **minimum permissions** needed (`PVEVMAdmin` is enough for VM provisioning).
- Rotate tokens periodically if used in CI/CD.

---

✅ With this, Terraform and any external scripts (like health checks) will use **tokens instead of root passwords**.

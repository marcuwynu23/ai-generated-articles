# 🛡️ How to Restore and Deploy HashiCorp Vault from Backup with Immutable Fix, NGINX Reverse Proxy, and Multi-VPS Setup

This guide walks through the complete process of restoring Vault from a backup, fixing immutability issues (`chattr`), configuring Vault on VPS1 with HTTPS and NGINX, and accessing it securely via Vault CLI from a second server (VPS2).

---

## 📦 Step 1: Restore Vault from Backup

Assume your backup ZIP file contains the following directory structure:

```
vault-backup.zip
├── core/
├── logical/
└── sys/
```

1. **Unzip the backup**:

```bash
unzip vault-backup.zip -d /opt/vault/data
```

2. **Fix immutability flags (if restored from an immutable source)**:

```bash
sudo chattr -iR /opt/vault/data
```

3. **Ensure proper ownership**:

```bash
sudo chown -R vault:vault /opt/vault/data
```

---

## 🔐 Step 2: Unseal Vault

If Vault was previously initialized, you’ll need the original **unseal keys**.

1. Set environment variable:

```bash
export VAULT_ADDR=https://127.0.0.1:8200
export VAULT_SKIP_VERIFY=true  # If using self-signed certs
```

2. Unseal:

```bash
vault operator unseal <unseal_key_1>
vault operator unseal <unseal_key_2>
vault operator unseal <unseal_key_3>
```

You’ll know it’s unsealed when:

```bash
vault status
# Sealed: false
```

---

## ⚙️ Step 3: Configure Vault on VPS1 with NGINX & HTTPS

### Vault Config File: `/etc/vault.d/vault.hcl`

```hcl
ui = true

storage "file" {
  path = "/opt/vault/data"
}

listener "tcp" {
  address       = "0.0.0.0:8200"
  tls_cert_file = "/opt/vault/tls/tls.crt"
  tls_key_file  = "/opt/vault/tls/tls.key"
}
```

### NGINX Config for `secrets.opensourceproject.org`

```nginx
server {
    listen 80;
    server_name secrets.opensourceproject.org;

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name secrets.opensourceproject.org;

    ssl_certificate /etc/letsencrypt/live/cert/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/cert/privkey.pem;

    location / {
        proxy_pass https://127.0.0.1:8200;
        proxy_ssl_verify off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

> 🔄 Reload NGINX:

```bash
sudo systemctl reload nginx
```

---

## 🌐 Step 4: Access Vault Remotely from VPS2

### Install Vault CLI (Client only) on VPS2:

```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt update && sudo apt install vault
```

### Configure Vault CLI:

```bash
export VAULT_ADDR=https://secrets.opensourceproject.org
export VAULT_SKIP_VERIFY=true  # Optional, if using self-signed certs
```

### Test connection:

```bash
vault status
```

If unsealed and token is valid, you’ll see `Sealed: false`.

### Use Vault:

```bash
vault login <your_token>
vault kv get secrets/aliasgen
```

---

## 🧯 Common Issues & Fixes

| Problem                             | Solution                                                    |
| ----------------------------------- | ----------------------------------------------------------- |
| `permission denied`                 | Ensure token has correct policies                           |
| `Connection refused` from NGINX     | Check if Vault is running and listening on `127.0.0.1:8200` |
| `tls: failed to verify certificate` | Use `VAULT_SKIP_VERIFY=true` or fix cert SANs               |
| Immutable errors                    | Run `sudo chattr -iR /opt/vault/data`                       |

---

## ✅ Final Tips

- 🛠 Always secure tokens and rotate when needed.
- 🧩 Use AppRole or other auth methods for production.
- 🔁 Regularly back up `/opt/vault/data`.
- 🧪 Test unseal and recovery periodically.

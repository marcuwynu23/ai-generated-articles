# 🖥️ How to Install and Configure Samba on Ubuntu

Samba is an open-source implementation of the SMB/CIFS networking protocol, allowing Linux systems to share files and printers with Windows and other clients.

This guide walks through installing Samba, configuring a share, creating users, and troubleshooting.

---

## 1. Update and Install Samba

First, update your package list and install Samba:

```bash
sudo apt update
sudo apt install samba -y
```

Check that the installation was successful:

```bash
smbd --version
```

---

## 2. Backup the Default Samba Configuration

Always back up the original configuration before making changes:

```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bkp
```

View the backup (optional):

```bash
cat /etc/samba/smb.conf.bkp
```

---

## 3. Configure Samba Share

Open the Samba configuration file:

```bash
sudo nano /etc/samba/smb.conf
```

At the bottom, add a new share definition:

```ini
[Share]
comment = Shared Folder
path = /
browseable = yes
writable = yes
guest ok = no
valid users = smbadmin
admin users = smbadmin
delete readonly = yes
inherit permissions = yes
create mask = 0664
directory mask = 0775
```

⚠️ **Important:**  
The `path = /` gives access to the root filesystem. For security, replace `/` with a specific directory, e.g., `/srv/samba/share`.

---

## 4. Test Configuration

Run the following command to validate the config file:

```bash
testparm
```

---

## 5. Create Samba User

Samba uses its own user accounts (mapped to system users).

1. Create a system account for Samba (no login shell, no home dir):
   ```bash
   sudo adduser --no-create-home --shell /usr/sbin/nologin smbadmin
   ```
2. Enable the Samba user:

   ```bash
   sudo smbpasswd -e smbadmin
   ```

3. Add this user to Samba:
   ```bash
   sudo smbpasswd -a smbadmin
   ```

---

## 6. Restart Samba Services

After making changes, restart Samba services:

```bash
sudo systemctl restart smbd nmbd
sudo systemctl enable smbd
```

Check service status:

```bash
sudo systemctl status smbd
```

---

## 7. Configure Firewall

Allow Samba through the firewall:

```bash
sudo ufw allow samba
sudo ufw status
```

You can confirm Samba is listening on port 445:

```bash
nmap -p 445 localhost
```

---

## 8. Access the Samba Share

From a Windows or Linux client, access the share using:

```
\\<server-ip>\Share
```

Log in with:

- **Username:** `smbadmin`
- **Password:** (the one you set with `smbpasswd`)

---

## ✅ Summary

- Installed Samba (`smbd` & `nmbd`)
- Backed up and edited `/etc/samba/smb.conf`
- Created a dedicated user `smbadmin`
- Tested configuration with `testparm`
- Restarted and enabled services
- Allowed Samba in the firewall

You now have a working Samba file server.

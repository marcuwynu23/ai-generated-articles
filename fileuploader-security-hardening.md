# 🔒 Securing a File Upload Microservice in Production

File uploads are one of the most **exploited attack vectors** on web
servers. If poorly configured, attackers can upload malicious scripts,
gain remote code execution, and even deface or take over the entire
server.

In this article, we'll break down a real-world scenario of a
Python/Flask file uploader service running with **systemd**, and show
step-by-step how to harden it for production use.

------------------------------------------------------------------------

## 🚨 The Problem: Unsafe Defaults

The original setup looked like this:

-   Service running as **root** (`User=root`, `Group=root`).\
-   Application code located inside `/root/production/...`.\
-   Upload directory (`/uploads`) owned by `root:nogroup`, with
    permissions `755`.\
-   Files saved with mode `644` (world-readable).\
-   Upload endpoints exposed publicly with **no authentication or rate
    limiting**.

**Why this is dangerous:** - If the uploader is compromised, the
attacker gets **root shell** (full system takeover).\
- World-readable `/uploads` allows any user to read files (data
leakage).\
- Without restrictions, someone could upload `.php`, `.py`, `.sh`, or
`.exe` and attempt remote execution.\
- No auth means *anyone* on the internet can fill your disk with junk
files.

------------------------------------------------------------------------

## ✅ Step 1: Run the Service as a Non-Root User

Create a dedicated system user for the service:

``` bash
sudo adduser --system --no-create-home --group fileuploader
```

Move your service code out of `/root` and into a proper directory:

``` bash
sudo mkdir -p /srv/fileuploader
sudo cp -r /root/production/micro-services/fileuploader/* /srv/fileuploader/
sudo chown -R fileuploader:fileuploader /srv/fileuploader
```

Update your **systemd unit**
(`/etc/systemd/system/fileuploader.service`):

``` ini
[Unit]
Description=File Uploader Service
After=network.target

[Service]
User=fileuploader
Group=fileuploader
WorkingDirectory=/srv/fileuploader
ExecStart=/usr/bin/python3 fileuploader.pyz
EnvironmentFile=/srv/fileuploader/.env
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

Reload and restart:

``` bash
sudo systemctl daemon-reload
sudo systemctl restart fileuploader
```

------------------------------------------------------------------------

## ✅ Step 2: Lock Down the Uploads Directory

Set proper ownership:

``` bash
sudo chown -R fileuploader:www-data /uploads
```

Restrict permissions:

``` bash
find /uploads -type d -exec chmod 750 {} \;
find /uploads -type f -exec chmod 640 {} \;
```

This ensures only the service and webserver group can access uploads ---
no "world read" anymore.

------------------------------------------------------------------------

## ✅ Step 3: Prevent Execution of Uploaded Files

Block any chance of uploaded scripts executing.

### Nginx config:

``` nginx
location /uploads {
    autoindex off;
    location ~* \.(php|py|sh|pl|cgi)$ {
        deny all;
    }
}
```

### Apache (`.htaccess` in /uploads):

``` apache
php_flag engine off
RemoveHandler .php .phtml .pl .py .sh
Options -ExecCGI
```

------------------------------------------------------------------------

## ✅ Step 4: Enforce File Restrictions

Inside your Flask app, whitelist only safe file types:

``` python
ALLOWED_EXTENSIONS = {'jpg', 'jpeg', 'png', 'pdf', 'docx', 'mp4'}

def allowed_file(filename):
    return '.' in filename and filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

file = request.files.get('file')
if not file or not allowed_file(file.filename):
    return jsonify({"error": "Invalid file type"}), 400
```

Limit upload size:

``` python
app.config['MAX_CONTENT_LENGTH'] = 50 * 1024 * 1024  # 50MB max
```

------------------------------------------------------------------------

## ✅ Step 5: Add Authentication to the API

Protect endpoints with an API key:

``` python
from flask import request, jsonify
import os

API_KEY = os.getenv("API_KEY")

@app.before_request
def require_api_key():
    if request.endpoint in ["file_uploader.upload_file", "file_uploader.upload_multiple_files"]:
        key = request.headers.get("X-API-KEY")
        if key != API_KEY:
            return jsonify({"error": "Unauthorized"}), 401
```

In your `.env`:

    API_KEY=supersecretkey123

Clients must now send:

    X-API-KEY: supersecretkey123

------------------------------------------------------------------------

## ✅ Step 6: Add Monitoring & Limits

-   Enable **fail2ban** to block repeated abusive requests.\
-   Monitor `/uploads` size with cron or a script to prevent disk
    exhaustion.\
-   Use reverse proxy rate limiting in Nginx/Apache.

Example Nginx limit:

``` nginx
limit_req_zone $binary_remote_addr zone=uploads:10m rate=5r/s;

location /api/bcloud/fileuploader/upload {
    limit_req zone=uploads burst=10 nodelay;
    proxy_pass http://127.0.0.1:2424;
}
```

------------------------------------------------------------------------

## 🔐 Final Checklist

-   [x] Service runs as `fileuploader` (not root).\
-   [x] Code in `/srv/fileuploader`, not `/root`.\
-   [x] `/uploads` restricted to `750/640`, owned by service user.\
-   [x] Script execution blocked in `/uploads`.\
-   [x] File type + size validation added.\
-   [x] Authentication (API key) required for uploads.\
-   [x] Monitoring & rate limiting in place.

------------------------------------------------------------------------

## ⚡ Conclusion

A file upload service is powerful but dangerous if left unprotected.\
By **removing root privileges, restricting file permissions, blocking
script execution, validating inputs, and adding auth**, you
significantly reduce the attack surface.

This hardening approach ensures your **uploads remain safe** and your
server **resilient against defacement, privilege escalation, and data
leaks**.

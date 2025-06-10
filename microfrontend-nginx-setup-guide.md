
# Hosting Multiple Micro Frontends on a Single Domain with NGINX

## 🌟 Overview

In modern web applications, **micro frontends** are a powerful architectural approach:
✅ Each team owns its own UI app  
✅ They’re independently built & deployed  
✅ Users see them as one seamless app!

**But how do you host them all together under one domain?** That’s where **NGINX** shines!

---

## 🔥 Scenario

Let’s say you have:  
- `App1`: **/app1/**  
- `App2`: **/app2/**  
- `App3`: **/app3/**  
Each built separately and output as static files.

Your goal:  
✅ **Single domain** (`test.cloudmateria.com`)  
✅ Clean, predictable URLs  
✅ No asset 404s or client-side routing issues

---

## ⚙️ Folder Structure

\`\`\`plaintext
/var/www/test/
├── app1/
│   ├── index.html
│   └── assets/
├── app2/
│   ├── index.html
│   └── assets/
└── app3/
    ├── index.html
    └── assets/
\`\`\`

---

## 📝 The NGINX Configuration

\`\`\`nginx
server {
  listen 443 ssl;
  server_name test.cloudmateria.com;

  ssl_certificate /etc/nginx/ssl/cloudflare.pem;
  ssl_certificate_key /etc/nginx/ssl/cloudflare.key;

  root /var/www/test;

  # === App1
  location = /app1 {
    return 301 https://$host/app1/;
  }
  location /app1/ {
    try_files $uri $uri/ /app1/index.html;
  }
  location /app1/assets/ {
    alias /var/www/test/app1/assets/;
  }

  # === App2
  location = /app2 {
    return 301 https://$host/app2/;
  }
  location /app2/ {
    try_files $uri $uri/ /app2/index.html;
  }
  location /app2/assets/ {
    alias /var/www/test/app2/assets/;
  }

  # === App3
  location = /app3 {
    return 301 https://$host/app3/;
  }
  location /app3/ {
    try_files $uri $uri/ /app3/index.html;
  }
  location /app3/assets/ {
    alias /var/www/test/app3/assets/;
  }

  # === Root fallback
  location = / {
    return 404;
  }
}
\`\`\`

---

## 🧩 How This Works

✅ **Trailing Slash Redirects**  
For example, \`https://test.cloudmateria.com/app1\` → automatically redirected to \`/app1/\`.

✅ **SPA Deep Linking**  
Each app uses:  
\`\`\`nginx
try_files $uri $uri/ /appX/index.html;
\`\`\`
This ensures **deep links** (like \`/app1/dashboard\`) fallback to the SPA’s entry point.

✅ **Static Asset Handling**  
Assets like \`/app1/assets/…\` are served using:  
\`\`\`nginx
location /app1/assets/ {
  alias /var/www/test/app1/assets/;
}
\`\`\`
**\`alias\`** ensures the correct absolute path.

---

## 🌟 Benefits of This Approach

✅ **Seamless user experience** — all micro frontends under one domain  
✅ **Independent deployments** — each app’s assets are isolated  
✅ **Secure** — HTTPS enforced  
✅ **Scalable** — easy to add \`/app4/\` in future!

---

## 🔥 Bonus: Build Configuration Tips

For this to work perfectly, ensure your **build tool** (like Vite, Webpack, or CRA) uses the correct **base path**.

✅ In Vite:  
\`\`\`js
export default defineConfig({
  base: '/app1/', // or /app2/ or /app3/
})
\`\`\`

✅ In React (CRA):  
\`\`\`json
"homepage": "/app1/"
\`\`\`

This ensures **assets like JS/CSS** are requested as \`/app1/assets/…\`, matching the NGINX config!

---

## 🚀 Summary

**With this setup, you’ve created:**  
- 🔥 A modern, modular frontend architecture  
- 🚀 Seamless deep-linking for each micro frontend  
- ⚡️ Blazing fast asset delivery — no broken links!

**Ready to deploy?**  
✅ Test NGINX:  
\`\`\`bash
sudo nginx -t
\`\`\`  
✅ Reload NGINX:  
\`\`\`bash
sudo systemctl reload nginx
\`\`\`

**Your users will thank you!** 🎉

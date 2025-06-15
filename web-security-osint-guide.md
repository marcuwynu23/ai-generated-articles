# 🔍 How to Perform OSINT on a Web Application Using WhatWeb and Wappalyzer — And Why Hiding Technologies Matters

## 🧠 Introduction: What Is OSINT in Web Applications?

**Open-Source Intelligence (OSINT)** refers to the gathering of information from publicly available sources. In the context of web applications, this means identifying technologies, frameworks, servers, and versions used in a target system — **without needing to penetrate it**.

While OSINT is often used by **ethical hackers and bug bounty hunters**, it is also leveraged by malicious actors looking for known vulnerabilities in **specific technology stacks or outdated components**.

---

## ⚠️ Why Hiding or Obfuscating Tech Stacks Is Crucial

When a web app exposes its:

- Framework (e.g., React, Angular, Laravel)
- Server (e.g., Nginx, Apache)
- CMS (e.g., WordPress, Drupal)
- Backend Language & Version (e.g., PHP 7.4)
- Libraries (e.g., jQuery 3.4.1, React Router 6.22.0)

…an attacker can **correlate that with known vulnerabilities** (CVEs), version exploits, or weaknesses in misconfiguration.

### 📌 Risks of Exposing Version Info:
- **React Router 5.x** had XSS issues if not configured properly.
- **Drupal 7** had multiple remote code execution CVEs.
- **PHP 7.4** is officially end-of-life (as of 2022).
- Server headers like `X-Powered-By`, `Server`, or even JavaScript hints can give away too much.

🧪 Obfuscation and minimal header exposure are **passive defenses** against automated reconnaissance.

---

## 🛠 Tools for OSINT Reconnaissance

There are many tools for OSINT gathering, but we’ll focus on two popular options:

- **WhatWeb (via Docker)**
- **Wappalyzer (Browser Extension)**

---

## 🧪 Method 1: Using WhatWeb via Docker

**WhatWeb** is a powerful reconnaissance tool that detects web technologies and server configurations.

### 📦 Install & Run WhatWeb:
```bash
docker run --rm secsi/whatweb https://your-target-domain.com -v
```

### ✅ What It Can Reveal:
- Server type (`Apache`, `cloudflare`, etc.)
- CMS (`Drupal`, `WordPress`)
- Backend language (`PHP`, `Python`)
- JavaScript libraries (if revealed in headers)
- Interesting HTTP headers:
  - `X-Powered-By`
  - `X-Generator`
  - `Server`
  - `X-Drupal-Cache`
  - `X-XSS-Protection`
  - and many others

---

## 🧪 Method 2: Using Wappalyzer (Browser Extension)

**Wappalyzer** is a browser extension for Chrome and Firefox that identifies technologies based on:

- HTTP headers
- HTML metadata
- JavaScript variables or global objects
- Webpack chunk content
- Service Workers and PWA info

### 📦 How to Use:
1. Install [Wappalyzer Extension](https://www.wappalyzer.com/)
2. Visit your target web app (e.g., `https://your-target-domain.com`)
3. Click the extension icon to view identified tech

### ✅ What It Can Reveal:
- Frontend Framework (React, Vue, Angular)
- JavaScript Libraries (jQuery, Lodash)
- CMS (Drupal, Joomla)
- PWA detection
- Analytics tools (Google Analytics, Facebook Pixel)

It even detects **version numbers**, especially from Webpack chunk files (like `vendor.js`).

---

## 🛡️ How to Hide or Obfuscate Your Stack

Here are key strategies to **minimize your OSINT exposure**:

### 1. **Strip Headers in Nginx**

```nginx
server {
    location / {
        proxy_hide_header X-Powered-By;
        proxy_hide_header Server;
        more_clear_headers 'X-Powered-By' 'Server' 'X-Generator';
    }
    add_header Server "hidden";
}
```

Requires `ngx_headers_more` module for `more_clear_headers`.

---

### 2. **Avoid Obvious JS File Names**

Instead of `vendor-react-router.js`:

```js
// Vite config
build: {
  rollupOptions: {
    output: {
      chunkFileNames: 'assets/[hash].js',
      entryFileNames: 'assets/[hash].js',
    }
  }
}
```

---

### 3. **Use Obfuscation Plugins**

For Vite/React:

```bash
npm install vite-plugin-obfuscator --save-dev
```

```ts
// vite.config.ts
import obfuscator from 'vite-plugin-obfuscator';

export default defineConfig({
  plugins: [obfuscator({
    controlFlowFlattening: true,
    stringArray: true,
    stringArrayEncoding: ['rc4'],
  })]
});
```

---

### 4. **Hide React Router Version**

Check for strings like `"react-router-dom": "6.22.0"` in your bundles. If found:

```ts
define: {
  '__REACT_ROUTER_VERSION__': '"-"',
}
```

Or strip with obfuscation plugins.

---

## ✅ Conclusion

Performing OSINT using tools like WhatWeb or Wappalyzer is simple, fast, and often the **first step attackers use in real-world intrusion attempts**. If your web application leaks technology details and version numbers:

- You’re giving away too much without a fight.
- It makes **automated vulnerability mapping easy**.
- It weakens your overall security posture.

➡️ Take the time to **strip headers**, **obfuscate code**, and **rename public JS files**. These simple steps can significantly reduce your web application's **attack surface**.

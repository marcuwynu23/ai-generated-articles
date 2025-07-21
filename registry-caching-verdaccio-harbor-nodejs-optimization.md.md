# 🚀 Full Guide to Optimizing Node.js Projects Using Verdaccio, Harbor, and Docker Cache

## 📌 Overview

This guide walks you through optimizing your Node.js project's build and deployment pipeline using:
- **Verdaccio**: Local NPM proxy and cache
- **Harbor**: Private container image registry
- **Docker Layer Caching**: To minimize redundant builds
- **Custom Build Images**: Pre-installed tools for fast Docker builds

---

## 1️⃣ Verdaccio – Local NPM Proxy and Cache

### 🔧 What is Verdaccio?
Verdaccio is a lightweight private proxy registry for NPM. It caches modules from the public NPM registry and allows publishing your own private packages.

### ✅ Why Use Verdaccio?
- Speeds up builds by caching packages locally.
- Reduces internet dependency and avoids npmjs.org rate limits.
- Enables reproducible builds by locking the source of dependencies.

### 🔌 Setup Verdaccio with Helm
```bash
helm upgrade --install verdaccio ./verdaccio-chart   --set service.type=NodePort   --set service.nodePort=30873   --set service.port=4873   --set verdaccio.config.web.title="Local Verdaccio"   --set verdaccio.config.web.publicUrl="http://192.168.1.118:30873"   --set persistence.enabled=true   --set persistence.size=5Gi   --set persistence.storageClass=standard
```

### 🔧 Configure NPM to Use Verdaccio
```bash
npm config set registry http://192.168.1.118:30873
```

---

## 2️⃣ Harbor – Internal Docker Registry

### 🔧 What is Harbor?
Harbor is a cloud-native container registry that stores, signs, and scans content.

### ✅ Why Use Harbor?
- Stores custom build images locally.
- Speeds up Docker image pulls within your network.
- Supports air-gapped and secure environments.

### 🛠️ Push Custom Image to Harbor
```bash
docker tag node-build-tools:18-slim 192.168.1.182:8030/library/node-build-tools:18-slim
docker push 192.168.1.182:8030/library/node-build-tools:18-slim
```

### 📦 Use Custom Image in Dockerfile
```dockerfile
FROM 192.168.1.182:8030/library/node-build-tools:18-slim AS build
```

---

## 3️⃣ Docker Layer Caching

### ✅ Best Practices for Faster Docker Builds
```dockerfile
COPY package*.json ./
RUN npm ci --prefer-offline --no-audit
COPY . .
```

- The `npm ci` step is cached unless `package.json` changes.
- Using `--prefer-offline` with Verdaccio allows cached resolution.
- `--no-audit` skips security audits, which are unnecessary in CI.

---

## 4️⃣ Prebuilt Node Build Image

### 🏗️ Dockerfile to Build Base Image
```dockerfile
FROM node:18-slim

RUN apt-get update && apt-get install -y \
    python3 make g++ bash

# Save this as: Dockerfile
# Then build and push:
# docker build -t node-build-tools:18-slim .
# docker tag node-build-tools:18-slim 192.168.1.182:8030/library/node-build-tools:18-slim
# docker push 192.168.1.182:8030/library/node-build-tools:18-slim
```

---

## 5️⃣ Production Build Example

### 📦 Dockerfile (Multi-stage Build)
```dockerfile
# --- Build Stage ---
FROM 192.168.1.182:8030/library/node-build-tools:18-slim AS build
ARG ENV=production

WORKDIR /app

COPY package*.json ./
RUN npm ci --prefer-offline --no-audit
COPY . .

RUN if [ "$ENV" = "staging" ]; then \
      npm run build:debug; \
      mv production/debug production/app; \
    else \
      npm run build:release; \
      mv production/release production/app; \
    fi

# --- Runtime Stage ---
FROM nginx:alpine
COPY --from=build /app/production/app /usr/share/nginx/html
COPY nginx/nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## 🧪 CI/CD Benefits

| Feature                        | Impact                                  |
|-------------------------------|------------------------------------------|
| Verdaccio NPM registry        | Speeds up install, enables offline builds |
| Harbor Docker registry        | Fast, secure image storage in LAN        |
| Custom base images            | Avoids redundant package installs        |
| Docker layer caching          | Only rebuilds changed layers             |

---

## 💡 Final Tips

- Use `.dockerignore` to reduce build context.
- Enable Docker BuildKit: `DOCKER_BUILDKIT=1 docker build ...`
- Cache node_modules in CI if possible.
- Periodically clean up Verdaccio and Harbor if disk space matters.

---

**Created:** July 21, 2025  
**Author:** Internal DevOps Guide  


# Setting Up an On-Premise Container Registry with Harbor and Configuring Kubernetes for Insecure HTTP Registry (Including containerd Configuration)

## Introduction

In this guide, we will set up **Harbor** as an **on-premise container registry** and configure **Kubernetes (K3s)** to work with an **insecure HTTP registry**. Additionally, we will configure **containerd** to allow pulling images from an insecure registry (HTTP) without TLS verification. This is particularly useful when working in private environments or during development when HTTPS may not be available.

**Prerequisites:**
- A Linux-based server (Ubuntu/Debian preferred).
- A working **K3s** Kubernetes cluster.
- Docker or **containerd** installed on the server.
- Basic knowledge of **Harbor**, **Docker**, **containerd**, and **Kubernetes**.

---

## Part 1: Setting Up Harbor (On-Premise Container Registry)

### Step 1: Install Docker

Ensure that **Docker** is installed on your Harbor server:

```bash
sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl enable --now docker
```

Verify Docker installation:

```bash
docker --version
```

### Step 2: Install Harbor

1. **Download Harbor**:

   Navigate to your desired directory and download the latest version of Harbor.

   ```bash
   cd /opt/harbor
   curl -L https://github.com/goharbor/harbor/releases/download/v2.3.3/harbor-offline-installer-v2.3.3.tgz -o harbor.tgz
   tar xvf harbor.tgz
   ```

2. **Configure Harbor**:

   Modify the `harbor.yml` configuration file to suit your needs (e.g., hostname, port, and credentials):

   ```bash
   cd /opt/harbor/harbor
   nano harbor.yml
   ```

   - Set the hostname to the internal IP or domain name of your Harbor server (`192.168.1.182`).
   - Configure the Harbor UI, database, and registry components as needed.

3. **Run the Installer**:

   Run the installer after configuring the `harbor.yml` file:

   ```bash
   sudo ./install.sh
   ```

4. **Verify the Harbor Setup**:

   Once the installation completes, you should be able to access the Harbor UI at `http://<your-server-ip>:80` from your browser.

---

## Part 2: Configuring Kubernetes for Insecure Registry (HTTP)

Kubernetes, when using **containerd**, requires special configuration to interact with insecure registries (HTTP instead of HTTPS).

### Step 1: Configure K3s for Insecure Registry

To configure **K3s** (with **containerd**) to allow pulling from an insecure registry, follow these steps:

1. **Create `registries.yaml`** Configuration File:

   On each worker node in your K3s cluster, create the configuration file:

   ```bash
   sudo nano /etc/rancher/k3s/registries.yaml
   ```

2. **Add the following content** to `registries.yaml`:

   ```yaml
   mirrors:
     "192.168.1.182:8030":
       endpoint:
         - "http://192.168.1.182:8030"

   configs:
     "192.168.1.182:8030":
       tls:
         insecure_skip_verify: true
   ```

   This configuration tells **K3s** to treat `192.168.1.182:8030` as an insecure registry and disable TLS verification.

3. **Restart K3s**:

   After configuring the `registries.yaml` file, restart the **K3s** service:

   ```bash
   sudo systemctl restart k3s
   ```

### Step 2: Create Docker Registry Secret in Kubernetes

Create an **image pull secret** in **Kubernetes** to authenticate with the insecure Harbor registry:

```bash
kubectl create secret docker-registry harbor-registry-secret   --docker-server=http://192.168.1.182:8030   --docker-username=marcuwynu23   --docker-password=Password123   --docker-email=marcuwynu23@gmail.com
```

### Step 3: Add `imagePullSecrets` in Kubernetes Deployment

In your **Kubernetes** deployment YAML file, specify the `imagePullSecrets`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: testw
spec:
  replicas: 1
  selector:
    matchLabels:
      app: testw
  template:
    metadata:
      labels:
        app: testw
    spec:
      containers:
      - name: testw
        image: "http://192.168.1.182:8030/bccs/bccs-gateway-staging:latest"
      imagePullSecrets:
      - name: harbor-registry-secret
```

### Step 4: Deploy the Pod/Deployment

Apply the deployment using:

```bash
kubectl apply -f testw-deployment.yaml
```

Verify the deployment:

```bash
kubectl get pods
```

---

## Part 3: Configuring containerd for Insecure Registry

In addition to configuring **K3s** to allow HTTP insecure registry access, we also need to configure **containerd** directly.

### Step 1: Create the `hosts.toml` for containerd

On each node in your K3s cluster (especially the worker nodes), configure **containerd** to allow the insecure registry.

1. **Create the directory for certificates** if it doesn't exist:

   ```bash
   sudo mkdir -p /etc/containerd/certs.d/192.168.1.182\:8030
   ```

2. **Create and edit `hosts.toml`**:

   ```bash
   sudo nano /etc/containerd/certs.d/192.168.1.182\:8030/hosts.toml
   ```

3. **Add the following content** to `hosts.toml`:

   ```toml
   [host."http://192.168.1.182:8030"]
     capabilities = ["pull", "resolve"]
     skip_verify = true
     plain_http = true
   ```

   This configuration:
   - Allows pulling and resolving images from the `http://192.168.1.182:8030` registry.
   - Skips TLS verification (`skip_verify = true`).
   - Forces containerd to use HTTP for the specified registry.

4. **Restart containerd**:

   After configuring `hosts.toml`, restart **containerd** to apply the changes:

   ```bash
   sudo systemctl restart containerd
   ```

5. **Add registry configuration to containerd config.toml**:

   Edit the `config.toml` file to add the insecure registry configuration:

   ```bash
   sudo nano /etc/containerd/config.toml
   ```

   Add the following configuration:

   ```toml
   [plugins."io.containerd.grpc.v1.cri".registry]
     config_path = "/etc/containerd/certs.d"

     [plugins."io.containerd.grpc.v1.cri".registry.configs."192.168.1.182:8030"]
       insecure = true
   ```

   This configuration directs **containerd** to use the insecure registry and skip verification.

6. **Restart containerd again**:

   After making the changes, restart **containerd** once more:

   ```bash
   sudo systemctl restart containerd
   ```

---

## Part 4: Accessing Harbor (Insecure Registry)

Once you’ve configured **Harbor**, **K3s**, and **containerd**, you can start using **Harbor** to store and manage your container images.

1. **Push images to Harbor**:

   Tag and push your Docker images to Harbor:

   ```bash
   docker tag your-image 192.168.1.182:8030/bccs/your-image:latest
   docker push 192.168.1.182:8030/bccs/your-image:latest
   ```

2. **Pull images from Harbor in Kubernetes**:

   When deploying pods or deployments in **Kubernetes**, the images will be pulled from **Harbor** automatically based on the `imagePullSecrets`.

---

## Troubleshooting

1. **Image Pull Errors**: If you encounter issues, check that:
   - The `registries.yaml` is correctly configured on all nodes.
   - The `imagePullSecrets` are set correctly in your Kubernetes deployment.
   - **Harbor** is running and accessible over HTTP.

2. **Pod Not Running**: Check the logs of your pod:

   ```bash
   kubectl logs <pod-name>
   ```

3. **Harbor Container Issues**: If **Harbor** containers are failing, check their logs:

   ```bash
   docker-compose logs
   ```

---

## Conclusion

By following these steps, you've successfully set up **Harbor** as your on-premise container registry and configured **K3s (containerd)** to access this registry over HTTP (insecure). You've also set up the necessary image pull secrets and registry configuration in Kubernetes.

This setup is useful for development and internal environments but always ensure to use secure (HTTPS) registries in production environments to protect your data.

Let me know if you have any questions or if anything is unclear!

# Automating SaaS Deployment Using VPS, Nginx, GitHub Actions, and Ansible

## Introduction

Deploying a SaaS application efficiently requires a robust, automated workflow. In this guide, we will set up a fully automated SaaS deployment pipeline using:

- **VPS** for hosting
- **NGINX** for reverse proxy
- **GitHub Actions** for CI/CD
- **Ansible** for configuration management and deployment
- **PM2** for process management
- **Certbot** for automatic SSL setup

We will configure **staging and production environments** with automatic deployments and rollback mechanisms.

---

## 1. Setting Up GitHub Secrets

To manage sensitive data securely, we will use GitHub Secrets. Go to **Settings > Secrets** in your repository and add the following secrets:

### **Production Secrets**

- `PROD_ENVIRONMENT` → (Paste the production `.env` file content)
- `PROD_VPS_IP` → (Your production server IP)
- `PROD_VPS_USER` → (SSH user for production)
- `PROD_REPO_URL` → (Git repo for production)

### **Staging Secrets**

- `STAGING_ENVIRONMENT` → (Paste the staging `.env` file content)
- `STAGING_VPS_IP` → (Your staging server IP)
- `STAGING_VPS_USER` → (SSH user for staging)
- `STAGING_REPO_URL` → (Git repo for staging)

---

## 2. Configuring GitHub Actions Workflow

Create a **GitHub Actions** workflow file: `.github/workflows/deploy.yml`.

```yaml
name: Deploy SaaS App

on:
  push:
    branches:
      - main
      - develop

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Install Ansible
        run: sudo apt update && sudo apt install -y ansible

      - name: Determine Environment
        id: set_env
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            echo "DEPLOY_ENV=production" >> $GITHUB_ENV
          else
            echo "DEPLOY_ENV=staging" >> $GITHUB_ENV
          fi

      - name: Set up Ansible Inventory
        run: |
          if [[ "$DEPLOY_ENV" == "production" ]]; then
            echo "[vps]" > inventory.ini
            echo "${{ secrets.PROD_VPS_IP }} ansible_user=${{ secrets.PROD_VPS_USER }} ansible_ssh_private_key_file=~/.ssh/id_rsa" >> inventory.ini
          else
            echo "[vps]" > inventory.ini
            echo "${{ secrets.STAGING_VPS_IP }} ansible_user=${{ secrets.STAGING_VPS_USER }} ansible_ssh_private_key_file=~/.ssh/id_rsa" >> inventory.ini
          fi

      - name: Deploy Application
        env:
          ENVIRONMENT: ${{ secrets.PROD_ENVIRONMENT }}
          REPO_URL: ${{ secrets.PROD_REPO_URL }}
        run: ansible-playbook -i inventory.ini ansible/deploy.yml
        if: github.ref == 'refs/heads/main'

      - name: Deploy to Staging
        env:
          ENVIRONMENT: ${{ secrets.STAGING_ENVIRONMENT }}
          REPO_URL: ${{ secrets.STAGING_REPO_URL }}
        run: ansible-playbook -i inventory.ini ansible/deploy.yml
        if: github.ref == 'refs/heads/develop'
```

---

## 3. Creating the Ansible Playbook

Create `ansible/deploy.yml` to handle both **production and staging** deployments.

```yaml
---
- name: Deploy SaaS Application
  hosts: vps
  become: yes
  vars:
    deploy_path: "/var/www/{{ lookup('env', 'APP_NAME') }}"
    backup_path: "{{ deploy_path }}_backup"

  tasks:
    - name: Ensure required directories exist
      file:
        path: "{{ item }}"
        state: directory
        owner: www-data
        group: www-data
        mode: "0755"
      loop:
        - "{{ deploy_path }}"
        - "{{ backup_path }}"

    - name: Backup current deployment
      shell: |
        rm -rf {{ backup_path }}/*
        cp -r {{ deploy_path }}/* {{ backup_path }}/
      when: lookup('fileglob', '{{ deploy_path }}/*', errors='ignore')

    - name: Clone repository
      git:
        repo: "{{ lookup('env', 'REPO_URL') }}"
        dest: "/tmp/{{ lookup('env', 'APP_NAME') }}"
        version: "{{ lookup('env', 'BRANCH') | default('main') }}"
        force: yes

    - name: Deploy new version
      synchronize:
        src: "/tmp/{{ lookup('env', 'APP_NAME') }}/"
        dest: "{{ deploy_path }}/"
        delete: yes
      register: sync_result
      ignore_errors: yes

    - name: Rollback on failure
      shell: |
        rm -rf {{ deploy_path }}/*
        cp -r {{ backup_path }}/* {{ deploy_path }}/
      when: sync_result.failed

    - name: Write environment variables
      copy:
        content: "{{ lookup('env', 'ENVIRONMENT') }}"
        dest: "{{ deploy_path }}/.env"
        owner: www-data
        group: www-data
        mode: "0600"

    - name: Restart Nginx
      systemd:
        name: nginx
        state: restarted
        enabled: yes

    - name: Restart application
      command: pm2 restart "{{ lookup('env', 'APP_NAME') }}"
      args:
        chdir: "{{ deploy_path }}"
      register: restart_result
      ignore_errors: yes

    - name: Restore from backup if restart fails
      shell: |
        rm -rf {{ deploy_path }}/*
        cp -r {{ backup_path }}/* {{ deploy_path }}/
      when: restart_result.failed
```

---

## 4. Setting Up Nginx with Certbot

Use the following **NGINX configuration template** (`ansible/templates/nginx.conf.j2`):

```nginx
server {
    listen 80;
    server_name {{ lookup('env', 'DOMAIN_NAME') }};
    location / {
        proxy_pass http://127.0.0.1:{{ lookup('env', 'APP_PORT') }};
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

To set up **SSL with Certbot**, run:

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com
```

---

## Conclusion

This setup provides a **fully automated SaaS deployment pipeline** that:

- Deploys to **staging** when pushing to `develop`
- Deploys to **production** when pushing to `main`
- Supports **rollback** on failure
- Automatically updates environment variables
- Uses **NGINX & Certbot** for SSL
- Ensures high availability with **PM2**

Now, you can scale your SaaS application with confidence! 🚀

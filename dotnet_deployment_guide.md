
# How to Deploy .NET on Ubuntu Linux

Deploying a .NET application on Ubuntu Linux involves several steps. Here’s a general guide to help you through the process:

## 1. Install .NET SDK/Runtime on Ubuntu

Before deploying your .NET application, you need to install the .NET SDK or Runtime on your Ubuntu server.

- **Install the Microsoft package signing key:**

  ```bash
  wget https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
  sudo dpkg -i packages-microsoft-prod.deb
  ```

- **Update the package index:**

  ```bash
  sudo apt-get update
  ```

- **Install the .NET SDK or Runtime:**

  - For .NET SDK (includes runtime):

    ```bash
    sudo apt-get install -y dotnet-sdk-7.0
    ```

  - For .NET Runtime only:

    ```bash
    sudo apt-get install -y dotnet-runtime-7.0
    ```

  *(Replace `7.0` with the desired version if you need a different one)*

## 2. Publish Your .NET Application

On your development machine, publish your .NET application for Linux.

- **Use the `dotnet publish` command:**

  ```bash
  dotnet publish -c Release -r ubuntu.20.04-x64 --self-contained
  ```

  This command creates a self-contained deployment, meaning the runtime is included with the application, which is useful if the target machine does not have the .NET runtime installed.

- If you don't want a self-contained deployment, use:

  ```bash
  dotnet publish -c Release -r ubuntu.20.04-x64 --no-self-contained
  ```

  Replace `ubuntu.20.04-x64` with the appropriate RID (Runtime Identifier) for your Ubuntu version.

## 3. Transfer Files to the Ubuntu Server

Transfer the published files to your Ubuntu server using SCP, SFTP, or any other preferred method.

- **Example using `scp`:**

  ```bash
  scp -r bin/Release/net7.0/ubuntu.20.04-x64/publish/ username@your-server-ip:/path/to/deployment/folder
  ```

## 4. Configure and Run the Application

- **SSH into your Ubuntu server:**

  ```bash
  ssh username@your-server-ip
  ```

- **Navigate to the directory where you copied your application files:**

  ```bash
  cd /path/to/deployment/folder
  ```

- **Give execute permissions to your application (if self-contained):**

  ```bash
  chmod +x YourAppName
  ```

- **Run your application:**

  ```bash
  ./YourAppName
  ```

  Or, if it's not self-contained, use:

  ```bash
  dotnet YourAppName.dll
  ```

## 5. (Optional) Configure as a Systemd Service

To ensure your application runs automatically and stays running, you can configure it as a `systemd` service.

- **Create a service file:**

  ```bash
  sudo nano /etc/systemd/system/yourapp.service
  ```

- **Add the following content to the file:**

  ```ini
  [Unit]
  Description=Your .NET Application

  [Service]
  WorkingDirectory=/path/to/deployment/folder
  ExecStart=/path/to/deployment/folder/YourAppName
  Restart=always
  RestartSec=10
  KillSignal=SIGINT
  SyslogIdentifier=yourapp
  User=www-data
  Environment=ASPNETCORE_ENVIRONMENT=Production

  [Install]
  WantedBy=multi-user.target
  ```

- **Reload systemd, start and enable your service:**

  ```bash
  sudo systemctl daemon-reload
  sudo systemctl start yourapp
  sudo systemctl enable yourapp
  ```

- **Check the status of your service:**

  ```bash
  sudo systemctl status yourapp
  ```

## 6. Configure Firewall and Reverse Proxy (Optional)

If your application is a web app, you may want to configure a reverse proxy like Nginx and open the necessary firewall ports.

- **Install Nginx:**

  ```bash
  sudo apt-get install nginx
  ```

- **Configure Nginx as a reverse proxy:**

  Create a new Nginx site configuration:

  ```bash
  sudo nano /etc/nginx/sites-available/yourapp
  ```

  Add the following content:

  ```nginx
  server {
      listen 80;
      server_name yourdomain.com;

      location / {
          proxy_pass http://localhost:5000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
      }
  }
  ```

- **Enable the site and restart Nginx:**

  ```bash
  sudo ln -s /etc/nginx/sites-available/yourapp /etc/nginx/sites-enabled/
  sudo systemctl restart nginx
  ```

## 7. Monitor and Maintain

Regularly check logs (`journalctl -u yourapp`) and monitor the application’s performance to ensure everything is running smoothly.

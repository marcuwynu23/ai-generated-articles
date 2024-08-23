
# How to Deploy a Node.js Application on Ubuntu VPS Using Forever and Nginx

Deploying a Node.js application on a VPS using `forever` is a simple and effective way to keep your application running in the background. This guide will walk you through the steps to deploy your Node.js application with `forever` and Nginx on an Ubuntu VPS.

## 1. Update Your Server

First, ensure that your server is up-to-date:

```bash
sudo apt-get update
sudo apt-get upgrade
```

## 2. Install Node.js and npm

You can install Node.js and npm using the NodeSource repository, which provides the latest versions.

### Install Node.js:

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```

This will install both Node.js and npm.

## 3. Install Your Node.js Application

If you haven’t already, you’ll need to set up your Node.js application on the server. You can clone it from your version control system (e.g., GitHub) or upload it directly.

### Example using Git:

```bash
git clone https://github.com/yourusername/yourproject.git
cd yourproject
```

### Install dependencies:

```bash
npm install
```

## 4. Install Forever

`forever` is a simple command-line tool for ensuring that a given script runs continuously. Install it globally using npm:

```bash
sudo npm install -g forever
```

## 5. Start Your Application with Forever

You can now start your Node.js application using `forever`:

```bash
forever start app.js
```

Replace `app.js` with the entry point to your application.

You can check the status of your application with:

```bash
forever list
```

To stop your application, use:

```bash
forever stop app.js
```

## 6. Set Up Nginx as a Reverse Proxy

To expose your Node.js application to the outside world, you can configure Nginx to act as a reverse proxy that forwards requests to your application.

### Install Nginx:

```bash
sudo apt-get install nginx
```

### Configure Nginx:

Create a new Nginx server block file:

```bash
sudo nano /etc/nginx/sites-available/yourapp
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name your_domain_or_IP;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

- Replace `your_domain_or_IP` with your domain name or server IP.
- Replace `3000` with the port your Node.js application is running on.

### Enable the Nginx configuration:

```bash
sudo ln -s /etc/nginx/sites-available/yourapp /etc/nginx/sites-enabled
```

### Test the Nginx configuration and restart the service:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

## 7. Allow Nginx through the Firewall

If you have a firewall enabled, you’ll need to allow Nginx traffic:

```bash
sudo ufw allow 'Nginx Full'
```

## 8. Monitor and Maintain

Your Node.js application should now be up and running, accessible through Nginx. You can monitor the status of your application with:

```bash
forever list
```

Check logs for any issues:

```bash
forever logs app.js
```

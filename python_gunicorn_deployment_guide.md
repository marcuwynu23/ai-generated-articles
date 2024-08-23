
# How to Deploy a Python Web Application Using Gunicorn on Ubuntu VPS

Deploying a Python web application on a VPS using Gunicorn is a common practice for serving Python applications. This guide will walk you through the steps to get your Python application up and running with Gunicorn on an Ubuntu VPS.

## 1. Update Your Server

First, ensure that your server is up-to-date:

```bash
sudo apt-get update
sudo apt-get upgrade
```

## 2. Install Python and Pip

Ensure that Python and Pip are installed on your server. You can install them using the following commands:

```bash
sudo apt-get install python3 python3-pip
```

## 3. Install and Set Up Your Python Application

If you haven’t already, you’ll need to set up your Python application on the server. You can clone it from your version control system (e.g., GitHub) or upload it directly.

### Example using Git:

```bash
git clone https://github.com/yourusername/yourproject.git
cd yourproject
```

### Install dependencies:

```bash
pip3 install -r requirements.txt
```

Ensure that your application entry point is properly defined (commonly `app.py` or `wsgi.py`).

## 4. Install Gunicorn

Gunicorn is a WSGI HTTP server that allows you to serve your Python application. Install it using pip:

```bash
pip3 install gunicorn
```

## 5. Test Gunicorn with Your Application

Before setting up a systemd service, it's a good idea to test Gunicorn to ensure it can serve your application:

```bash
gunicorn --bind 0.0.0.0:8000 wsgi:app
```

Replace `wsgi:app` with the module and callable name that points to your application. For example, if your application entry point is `app.py` and contains `app = Flask(__name__)`, you would use `app:app`.

Your application should now be accessible via your server's IP address on port 8000.

## 6. Set Up Gunicorn as a Systemd Service

To ensure that your application starts on boot and restarts if it fails, you can set up Gunicorn as a systemd service.

### Create a systemd service file:

```bash
sudo nano /etc/systemd/system/yourapp.service
```

### Add the following content to the file:

```ini
[Unit]
Description=Gunicorn instance to serve yourapp
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/path/to/yourapp
ExecStart=/usr/local/bin/gunicorn --workers 3 --bind unix:yourapp.sock -m 007 wsgi:app

[Install]
WantedBy=multi-user.target
```

- Replace `/path/to/yourapp` with the path to your application directory.
- Replace `yourapp.sock` with the name you want for your socket file.
- Replace `wsgi:app` with the module and callable name of your application.

### Start and enable the Gunicorn service:

```bash
sudo systemctl start yourapp
sudo systemctl enable yourapp
```

## 7. Configure Nginx as a Reverse Proxy

To expose your application to the outside world, you can configure Nginx to act as a reverse proxy that forwards requests to Gunicorn.

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
        include proxy_params;
        proxy_pass http://unix:/path/to/yourapp/yourapp.sock;
    }
}
```

- Replace `your_domain_or_IP` with your domain name or server IP.
- Replace `/path/to/yourapp/yourapp.sock` with the path to your Gunicorn socket.

### Enable the Nginx configuration:

```bash
sudo ln -s /etc/nginx/sites-available/yourapp /etc/nginx/sites-enabled
```

### Test the Nginx configuration and restart the service:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

## 8. Allow Nginx through the Firewall

If you have a firewall enabled, you’ll need to allow Nginx traffic:

```bash
sudo ufw allow 'Nginx Full'
```

## 9. Monitor and Maintain

Your Python application should now be up and running, accessible through Nginx. You can monitor the status of your Gunicorn service with:

```bash
sudo systemctl status yourapp
```

Check logs for any issues:

```bash
journalctl -u yourapp
```

# How to Use Monit for Monitoring a Web Application on Linux


How to Use Monit for Monitoring a Web Application on Linux

Monit is a lightweight and easy-to-use monitoring tool for Unix-based systems that helps monitor and manage processes, services, filesystems, and more. In this article, we'll cover how to use Monit to monitor a web application, detect downtime or high load, and set up email notifications for alerts.

Step 1: Install Monit on Your Linux System
Monit can be installed via the package manager for most Linux distributions. Here's how to install it on Ubuntu and CentOS.

On Ubuntu/Debian:
    sudo apt-get update
    sudo apt-get install monit

On CentOS/RHEL:
    sudo yum install monit

Step 2: Configure Monit
Once Monit is installed, you need to configure it to monitor your web application. Monit’s configuration file is located at /etc/monit/monitrc.

Step 2.1: Open the Monit Configuration File
    sudo nano /etc/monit/monitrc

Step 2.2: Configure Basic Settings
Uncomment the following lines to set up a basic Monit configuration. This includes enabling the web interface and configuring email notifications.

    # Set Monit to check the system at 30-second intervals
    set daemon 30

    # Enable the web interface
    set httpd port 2812 and
        use address localhost  # Only allow connections from localhost
        allow localhost        # Allow localhost access without authentication

    # Configure the email notification settings
    set mailserver smtp.example.com port 587
        username "your-email@example.com" password "your-password"
        using tlsv1

    # Define the recipient email for alerts
    set alert your-email@example.com

Replace the smtp.example.com with your SMTP server details (e.g., Gmail, SendGrid) and add your email credentials. This will enable Monit to send email notifications when an alert is triggered.

Step 2.3: Configure Monit to Monitor Your Web Application
Now we will configure Monit to monitor your web application. We’ll check if the web server is up, restart it if it goes down, and set up an alert for high load.

Here’s an example configuration to monitor an Nginx-based web application running on port 80:

    # Check if the web server is running
    check host mywebapp with address example.com
        if failed port 80 protocol http then alert
        if failed port 80 protocol http
            with timeout 15 seconds for 3 cycles then restart
        if loadavg (1min) > 4 then alert

In this configuration:
- Monit checks if the web application is accessible on example.com at port 80 using HTTP.
- If Monit detects that the web server is down, it will try to restart the web server after 3 failed checks.
- An alert is triggered if the load average exceeds 4 within 1 minute.

Step 3: Set Up Email Notifications
To receive email notifications, you’ll need to configure Monit to send alerts when specific conditions are met (like downtime or high load). We’ve already configured the alert line in the configuration file to send alerts to your-email@example.com.

To verify email settings, use the following block:

    set mail-format {
      from: monit@your-domain.com
      subject: Monit alert -- $SERVICE $EVENT
      message: $SERVICE $EVENT at $DATE on $HOST: $DESCRIPTION.
    }

Step 4: Enable and Start Monit
After configuring Monit, you need to start the service and enable it to run at boot.

Enable and Start Monit:
    sudo systemctl enable monit
    sudo systemctl start monit

Check Monit Status:
    sudo systemctl status monit

Step 5: Access the Monit Web Interface
Monit comes with a simple web interface that allows you to view the status of monitored services and hosts.

To access the Monit web interface, open your browser and go to http://localhost:2812. If you configured Monit to bind to your server’s IP address, you can access it remotely (e.g., http://your-server-ip:2812).

By default, the web interface is configured to allow access from localhost only. If you need to access it remotely, modify the use address line in the Monit configuration and replace localhost with the server’s IP address.

Step 6: Verify Configuration
To verify that Monit is correctly monitoring your web application, use the following command:

    sudo monit status

This will display the current status of all the monitored services, including whether Monit is able to reach your web application.

Step 7: Testing Monit
- Test Web Application Monitoring: Stop the web server manually to see if Monit detects the failure and sends an email notification.

    sudo systemctl stop nginx

    Monit should send an email alert, attempt to restart the server, and notify you again if it successfully restarted the service.

- Test High Load Monitoring: Simulate high load on the server and check if Monit triggers an alert.

Conclusion
Monit is an excellent tool for monitoring web applications, and it's simple to set up and use. By following this guide, you now have Monit monitoring your web application, checking for downtimes, and sending email alerts when issues arise. Monit can also automatically attempt to restart failed services, providing a first layer of self-healing for your web infrastructure.

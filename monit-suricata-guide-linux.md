
# How to Monitor Suricata Using Monit and Send Email Alerts

## Overview
Monit is a lightweight and easy-to-use monitoring tool that can be used to monitor processes like Suricata and ensure they are always running. In this guide, we will configure Monit to monitor Suricata, detect when it is down or using too many resources, and send email alerts when necessary.

## Step 1: Install and Configure Monit

### Install Monit
On Ubuntu/Debian systems:
```bash
sudo apt-get update
sudo apt-get install monit
```

On CentOS/RHEL systems:
```bash
sudo yum install monit
```

## Step 2: Configure Monit to Monitor Suricata

1. Open Monit’s configuration file:
   ```bash
   sudo nano /etc/monit/monitrc
   ```

2. Add a configuration block to monitor the **Suricata** process. This block will check if Suricata is running, restart it if necessary, and alert if there’s a problem.

   Add the following lines to the `monitrc` file:
   ```bash
   check process suricata with pidfile /var/run/suricata.pid
     start program = "/usr/bin/systemctl start suricata"
     stop program = "/usr/bin/systemctl stop suricata"
     if cpu > 60% for 2 cycles then alert
     if memory > 500 MB for 2 cycles then alert
     if 5 restarts within 5 cycles then timeout
   ```

   This configuration will:
   - **Monitor the Suricata process**.
   - **Restart Suricata** if it crashes.
   - **Send email alerts** if Suricata consumes too much CPU or memory.

## Step 3: Configure Email Alerts in Monit

Monit needs to be configured to send emails when alerts are triggered.

1. In the `monitrc` file, configure the email server settings. Add the following lines:
   
   ```bash
   set mailserver smtp.example.com port 587
     username "your-email@example.com" password "your-email-password"
     using tlsv1  # Use TLS for secure email delivery

   set alert your-email@example.com  # Your email to receive alerts

   # Optional: Customize the email content
   set mail-format {
     from: monit@your-domain.com
     subject: Monit alert -- $SERVICE $EVENT
     message: $SERVICE $EVENT at $DATE on $HOST: $DESCRIPTION.
   }
   ```

   Replace:
   - `smtp.example.com` with the SMTP server of your email provider (e.g., Gmail, SendGrid, or your own email server).
   - `your-email@example.com` with your actual email address for sending the email.
   - `your-email-password` with the password for the SMTP account.

## Step 4: Enable and Start Monit

After configuring Monit, enable the service and start it.

1. Enable Monit to start at boot:
   ```bash
   sudo systemctl enable monit
   ```

2. Start the Monit service:
   ```bash
   sudo systemctl start monit
   ```

3. Check the status of Monit to ensure it is running:
   ```bash
   sudo systemctl status monit
   ```

4. You can also check Monit’s internal status with the command:
   ```bash
   sudo monit status
   ```

## Step 5: Test Email Alerts

1. **Simulate a Suricata Failure**: To test if Monit sends an email notification, stop the Suricata service manually and see if Monit detects it and sends an email alert.

   Stop Suricata:
   ```bash
   sudo systemctl stop suricata
   ```

   Wait for Monit to detect the failure and check your email for the alert.

2. **Check Logs**: You can also check Monit logs to see if the alert was triggered:
   ```bash
   tail -f /var/log/monit.log
   ```

## Step 6: Access the Monit Web Interface

If you want to access Monit via the web interface, you can enable the web interface in the `monitrc` file:

1. Open the Monit configuration file:
   ```bash
   sudo nano /etc/monit/monitrc
   ```

2. Uncomment or add the following lines to enable the web interface:
   ```bash
   set httpd port 2812 and
     use address localhost  # Only allow localhost
     allow localhost        # Allow localhost access without authentication
   ```

3. Access the Monit web interface by opening your browser and navigating to:
   ```
   http://localhost:2812
   ```

   You can view Suricata’s status and other monitored services here.

## Conclusion

By using Monit, you can easily monitor Suricata’s health and performance, and set up email alerts to notify you when Suricata is down, consuming too many resources, or needs to be restarted. Monit is a lightweight yet powerful tool that ensures your Suricata-based IDS/IPS system remains operational, providing you with peace of mind and quick recovery if issues arise.

Step 1: Set Up Active Directory Domain Services (AD DS) on Windows Server
Install Windows Server:

Make sure you have a Windows Server edition installed. You can use Windows Server 2016, 2019, or 2022 for this setup.
Promote the Server to a Domain Controller:

First, you need to promote the server to become the Domain Controller (DC). This allows the server to manage Active Directory.
Open Server Manager (usually, it opens automatically after logging into the server).
In Server Manager, click on "Manage" in the top-right corner and then select "Add Roles and Features".
Proceed through the wizard until you reach the "Roles" section.
Select "Active Directory Domain Services" and click Next to continue.
Follow the wizard through the installation, and once done, you will need to promote the server.
Promote the Server to a Domain Controller:

After installation, you'll see a notification in the Server Manager saying that the server needs to be promoted.
Click on "Promote this server to a domain controller".
Follow the wizard and select the following:
Add a new forest (if this is the first domain controller in the organization) or select Add a domain controller to an existing domain if you're adding it to an existing domain.
Set the root domain name (for example, example.com).
Set the Directory Services Restore Mode (DSRM) password.
Complete the Promotion:

After configuring everything, click Next, review the settings, and then click Install. The server will automatically restart, and it will now be a Domain Controller for the Active Directory domain you created.
Step 2: Configure Active Directory Users and Computers
Once the Domain Controller is set up, you can start managing users, groups, and organizational units.

Access Active Directory Users and Computers:

On the Domain Controller, open Server Manager.
Click Tools > Active Directory Users and Computers.
Create Users:

In the Active Directory Users and Computers window, right-click on Users (or another Organizational Unit if you've set one up) and select New > User.
Fill out the required fields (e.g., first name, last name, username) and set a password for the user.
Click Next and then Finish.
Create Groups:

Similarly, you can right-click on Users or another Organizational Unit, and select New > Group to create security or distribution groups.
Step 3: Set Up the Client Machine to Join the Domain
Now that you have your AD server running, the next step is to join a client machine (e.g., a Windows 10 or 11 computer) to the domain.

Configure Network Settings on the Client Machine:

Ensure the client machine has a static IP address or DHCP set up properly.
Set the DNS server on the client to point to the IP address of the Domain Controller (your AD server).
Join the Client Machine to the Domain:

On the client machine, open Settings > System > About.
Scroll down to Related settings and click System info.
In the System window, click Change settings next to Computer name, domain, and workgroup settings.
In the Computer Name/Domain Changes window, click Change.
In the Member of section, select Domain and type the domain name you configured earlier (e.g., example.com).
Click OK.
Enter Domain Credentials:

A prompt will appear asking for credentials. Enter the username and password of a domain administrator (this is the account created when you set up the AD DS server).
If everything is correct, you’ll receive a message saying "Welcome to the domain".
Restart the Client Machine:

After joining the domain, the client machine will need to restart.
After the restart, you can log in using domain credentials (e.g., domain\username).
Step 4: Verify Client Connection to Domain
Log In to the Client:

After the client machine restarts, log in with the domain credentials you created earlier.
On the login screen, ensure you select the domain from the drop-down list (it should show something like example.com).
Verify in Active Directory:

Go back to Active Directory Users and Computers on the Domain Controller.
You should see the client machine listed under Computers or in the Organizational Unit (OU) you have set up.
Step 5: Troubleshooting
DNS Configuration: Make sure that the client machine points to the Domain Controller as the DNS server. Active Directory relies heavily on DNS to locate domain controllers.
Firewall Issues: Ensure that any firewalls on the Domain Controller and the client machine allow the necessary traffic for Active Directory to function properly (such as DNS, Kerberos, and LDAP).

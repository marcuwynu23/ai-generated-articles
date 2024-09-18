
# How to Set Up Passwordless SSH Using `authorized_keys`

Secure Shell (SSH) is a widely used protocol for securely accessing remote systems. By default, SSH requires users to authenticate using a password. However, using SSH keys allows you to log into remote servers without repeatedly entering passwords, offering both convenience and enhanced security. In this article, we’ll walk through the steps to set up passwordless SSH access using the `authorized_keys` file.

---

### Why Use SSH Keys?

SSH key-based authentication is preferred over password authentication for several reasons:

1. **Security**: SSH keys are much harder to brute force than passwords.
2. **Convenience**: Once set up, SSH keys allow you to log in without having to enter a password each time.
3. **Automation**: SSH keys are essential for automating processes like secure file transfers and remote command execution.

---

### Step 1: Generate an SSH Key Pair

If you don’t already have an SSH key pair, you can easily generate one using the `ssh-keygen` command.

#### Command to Generate SSH Key Pair:

```bash
ssh-keygen -t rsa -b 4096
```

Here’s a breakdown of the command:
- **`-t rsa`**: Specifies the type of key to create (RSA).
- **`-b 4096`**: Creates a 4096-bit key for stronger security.

You will be prompted with the following questions:
- **File location**: By default, the key is stored in `~/.ssh/id_rsa`. You can press `Enter` to accept the default.
- **Passphrase**: You can enter a passphrase for extra security, but this step is optional. If you don’t want a passphrase, just press `Enter` twice.

#### Verify Key Generation:
After generating the keys, you will have two files:
- **`id_rsa`**: Your private key (keep this secure).
- **`id_rsa.pub`**: Your public key (to be shared with the remote server).

---

### Step 2: Copy the Public Key to the Remote Server

The next step is to transfer your public key (`id_rsa.pub`) to the remote server. The easiest way to do this is by using the `ssh-copy-id` command, which automates the process of adding your public key to the remote server's `authorized_keys` file.

#### Using `ssh-copy-id`:

```bash
ssh-copy-id user@remote_host
```

Replace `user` with your remote server’s username and `remote_host` with the server’s IP address or hostname.

This command does the following:
- It connects to the remote server using SSH.
- It appends your public key to the `~/.ssh/authorized_keys` file on the remote server.
- If needed, it creates the `.ssh` directory on the remote server.

#### Alternative: Manually Copy the Key

If `ssh-copy-id` is not available, you can manually copy the key to the server.

##### Step-by-Step Instructions:
1. **Display your public key**:
   On your local machine, run:
   ```bash
   cat ~/.ssh/id_rsa.pub
   ```
   Copy the output of the public key.

2. **SSH into the remote server**:
   ```bash
   ssh user@remote_host
   ```

3. **Create the `.ssh` directory (if it doesn’t exist)**:
   ```bash
   mkdir -p ~/.ssh
   ```

4. **Append the public key to the `authorized_keys` file**:
   ```bash
   echo "your_public_key_here" >> ~/.ssh/authorized_keys
   ```

5. **Set the correct permissions**:
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

The above commands ensure that the `.ssh` directory has the correct permissions and that the `authorized_keys` file is readable only by the owner.

---

### Step 3: Set Correct Permissions

For SSH to work properly, the permissions on the `.ssh` directory and the `authorized_keys` file must be correct. Ensure the following permissions on the remote server:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

- **`~/.ssh` directory**: Should have `700` permissions, which means only the owner can read, write, and execute.
- **`~/.ssh/authorized_keys` file**: Should have `600` permissions, meaning only the owner can read and write.

---

### Step 4: Test the SSH Connection

Now, test if you can log into the remote server without a password:

```bash
ssh user@remote_host
```

If everything is set up correctly, you should be able to log in without entering a password. The SSH connection will use the key you just configured for authentication.

---

### Step 5: Troubleshooting

If passwordless login doesn’t work, check the following:

#### 1. **SSH Service is Running**
Ensure the SSH service is running on the remote server:

```bash
sudo systemctl status sshd
```

#### 2. **Check `sshd_config` File**

The SSH server configuration file (`/etc/ssh/sshd_config`) must allow key-based authentication. Check the following settings:

```bash
sudo nano /etc/ssh/sshd_config
```

Ensure the following lines are set:

```
PubkeyAuthentication yes
AuthorizedKeysFile     %h/.ssh/authorized_keys
```

After making any changes, restart the SSH service:

```bash
sudo systemctl restart sshd
```

#### 3. **File Permissions**
Ensure that both the `.ssh` directory and the `authorized_keys` file on the remote server have the correct permissions (`700` for the directory and `600` for the file).

---

### Why Use SSH Key-Based Authentication?

- **Security**: SSH keys provide stronger protection than passwords. With a strong key pair, brute force attacks are virtually impossible.
- **Convenience**: Once configured, SSH keys eliminate the need to enter a password each time you connect to a remote system.
- **Automation**: SSH keys are essential for automation in DevOps and system administration tasks, enabling secure, automated access for scripts and services.

---

### Conclusion

Using SSH keys for passwordless authentication is a simple yet powerful way to improve your security and streamline your workflow. Once set up, it enables you to access remote systems quickly and securely without the hassle of entering passwords. Follow the steps outlined in this guide to set up your SSH keys and enjoy a safer, more efficient way to manage your servers.

By ensuring your permissions are correct and your SSH server is properly configured, you can confidently set up passwordless SSH access across multiple systems, making system administration tasks much easier.

---
title: "Complete Guide: Securing SSH and Using Private Keys on Linux"
date: 2026-03-04
categories: [Linux, Security, SSH]
tags: [ssh, linux, security, private-keys, hardening, sysadmin]
author: gnolasco
---

Secure Shell (SSH) is the standard protocol for remotely managing Linux systems. While password authentication works, it is significantly less secure than key-based authentication. Using SSH private keys improves security, prevents brute-force attacks, and enables safer automation for administrators.

This guide walks through generating SSH keys, configuring servers for key-based authentication, and applying SSH hardening best practices.

## Prerequisites

- A Linux client machine
- Access to a remote Linux server
- A user account with `sudo` privileges on the server
- OpenSSH installed (default on most Linux systems)

---

# 1. Generate an SSH Key Pair

SSH authentication uses a **public/private key pair**.

- **Private key** → stays on your local machine and must never be shared  
- **Public key** → stored on the remote server

Generate a modern SSH key:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Recommended options:

- `-t ed25519` → modern and secure algorithm
- `-C` → adds a comment to identify the key

When prompted:

- Choose a **secure passphrase** (recommended)
- Accept the default location:

```
~/.ssh/id_ed25519
```

Files created:

```
~/.ssh/id_ed25519      # Private key (KEEP SECRET)
~/.ssh/id_ed25519.pub  # Public key
```

---

# 2. Copy the Public Key to the Remote Server

The easiest way to install your key is using `ssh-copy-id`.

```bash
ssh-copy-id user@server-ip
```

Example:

```bash
ssh-copy-id admin@192.168.1.50
```

This command will:

- Create the `.ssh` directory if it does not exist
- Add your key to:

```
~/.ssh/authorized_keys
```

You can now log in using SSH:

```bash
ssh user@server-ip
```

---

# 3. Verify Key Authentication

Test the connection:

```bash
ssh user@server-ip
```

If everything is configured correctly:

- You will **not be asked for the account password**
- Authentication will use your SSH key

For debugging:

```bash
ssh -v user@server-ip
```

The `-v` flag shows detailed connection information.

---

# 4. Harden the SSH Server Configuration

Once key authentication works, disable insecure login methods.

Edit the SSH configuration:

```bash
sudo nano /etc/ssh/sshd_config
```

Recommended settings:

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
PermitEmptyPasswords no
MaxAuthTries 3
```

Explanation:

| Setting | Description |
|------|------|
| PermitRootLogin no | Prevent direct root login |
| PasswordAuthentication no | Disable password logins |
| PubkeyAuthentication yes | Allow key-based authentication |
| MaxAuthTries 3 | Limit authentication attempts |

Restart SSH:

```bash
sudo systemctl restart ssh
```

⚠️ **Important:**  
Verify SSH key login works before disabling password authentication to avoid locking yourself out.

---

# 5. Set Correct SSH Permissions

SSH requires strict file permissions.

Apply the correct permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
```

Expected permissions:

| File | Permission |
|----|----|
| `~/.ssh/` | 700 |
| `authorized_keys` | 600 |
| private keys | 600 |

Incorrect permissions may cause SSH authentication failures.

---

# 6. Use SSH Agent for Key Management

If you use a passphrase, the SSH agent can store your decrypted key for the session.

Start the agent:

```bash
eval "$(ssh-agent -s)"
```

Add your key:

```bash
ssh-add ~/.ssh/id_ed25519
```

Now your key will be cached securely during the session.

---

# 7. Optional Security Enhancements

## Change the Default SSH Port

Changing the SSH port reduces automated scanning.

Edit:

```
/etc/ssh/sshd_config
```

Example:

```
Port 2222
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

Connect using:

```bash
ssh -p 2222 user@server-ip
```

---

## Install Fail2Ban

Fail2Ban blocks IP addresses after repeated failed login attempts.

Install on Ubuntu/Debian:

```bash
sudo apt install fail2ban -y
```

Enable and start the service:

```bash
sudo systemctl enable --now fail2ban
```

---

# 8. Backup Your SSH Keys

Your private key is required to access servers.

Best practices:

- Keep **secure backups**
- Store keys in **encrypted password managers**
- Avoid copying private keys across many devices

Never share files such as:

```
id_rsa
id_ed25519
```

Anyone with access to these files can log in to your servers.

---

# Conclusion

SSH key authentication is one of the most important security improvements for Linux servers. By replacing passwords with private keys and applying basic SSH hardening practices, you dramatically reduce the risk of brute-force attacks and unauthorized access.

Key takeaways:

- Use **modern SSH keys (ed25519)**
- Disable password authentication
- Protect private keys with passphrases
- Enforce proper file permissions
- Implement additional protections like **Fail2Ban**

Securing SSH is a fundamental skill for any Linux administrator managing remote systems.

🔐 Secure your servers and manage them with confidence.

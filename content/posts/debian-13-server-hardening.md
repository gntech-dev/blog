---
title: "Advanced Debian 13 Server Hardening: A Practical Blueprint"
date: 2026-05-01T21:24:00-04:00
draft: false
tags: [ "security", "sysadmin", "debian", "hardening", "linux" ]
description: "A step-by-step, production-grade hardening guide for Debian 13 servers, with automation and auditing in mind."
---

Deploying a Debian 13 server? Here’s a battle-tested hardening blueprint for robust, secure, and auditable servers.

## 1. Full System Upgrade & Minimal Install

- Install only what you need. Remove unneeded packages, documentation, games, and sample configs.
- Update everything at once:
  ```bash
  sudo apt update && sudo apt full-upgrade -y && sudo apt autoremove --purge -y
  ```

## 2. User & SSH Lockdown

- Require SSH key authentication. Disable password auth and root login.
- Change default SSH port and restrict by IP.
- Use `sshguard` or `fail2ban` for brute-force throttling.

```bash
sudo nano /etc/ssh/sshd_config
# Set:
PermitRootLogin no
PasswordAuthentication no
Port 443    # Example alternative port

# Allow only specific users:
AllowUsers youradminuser

sudo systemctl reload ssh
```

## 3. Strict Firewall Rules & Network Hardening

**UFW (simple)**
```bash
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw allow <ssh-port>/tcp comment 'SSH'
sudo ufw enable
```

**Advanced: nftables**

- Replace iptables/ufw for production servers.
- Example for SSH, HTTP/HTTPS only:

```bash
sudo apt install nftables
sudo systemctl enable --now nftables

sudo tee /etc/nftables.conf > /dev/null <<EOF
#!/usr/sbin/nft -f

table inet filter {
  chain input {
    type filter hook input priority 0;
    ct state established,related accept
    iif "lo" accept
    ip saddr <your-ip-address>/32 tcp dport <ssh-port> accept
    tcp dport {80, 443} accept
    counter drop
  }
}
EOF

sudo systemctl reload nftables
```

## 4. Mandatory 2FA for SSH

- Use `google-authenticator` or `pam-u2f` for two-factor logins.
- Enforce with PAM (`libpam-google-authenticator` or `libpam-u2f`).

## 5. Harden Kernel & System Parameters

Edit `/etc/sysctl.conf` and apply:

```conf
# IPv4/IPv6 hardening
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0

# Disable IPv6 if not used
net.ipv6.conf.all.disable_ipv6 = 1
```
Apply with `sudo sysctl -p`.

## 6. Automatic Patch Management

- Install and configure auto-updates:
  ```bash
  sudo apt install unattended-upgrades apt-listchanges apticron
  sudo dpkg-reconfigure --priority=low unattended-upgrades
  ```
- Optional: set up root email notification for failed updates.

## 7. Intrusion Detection & File Integrity

- Deploy AIDE or OSSEC for tripwire-style file integrity.
  ```bash
  sudo apt install aide
  sudo aideinit
  sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db
  ```
- Automate checks daily via cron or systemd timer.

## 8. Audit Logging

- Enable auditd for tracking sensitive changes:
  ```bash
  sudo apt install auditd audispd-plugins
  sudo systemctl enable --now auditd
  ```
- Monitor: auth.log, audit.log, kernel logs.

## 9. Hardened Service Configuration

- Run public-facing apps in systemd sandboxes or Docker.
- Use minimum capabilities and AppArmor/Mandatory Access Control (`sudo apt install apparmor apparmor-profiles`).

## 10. Extra: Scripts For Repeatable Hardening

Automate these steps. Example GitHub repo: [debian-hardening-script](https://github.com/konstruktoid/hardening)

_Sample script:_
```bash
#!/bin/bash

useradd -m -G sudo secureduser
sed -i '/^PermitRootLogin/s/yes/no/' /etc/ssh/sshd_config
sed -i '/^PasswordAuthentication/s/yes/no/' /etc/ssh/sshd_config
systemctl reload ssh
apt install -y ufw fail2ban auditd aide apparmor
ufw default deny incoming
ufw allow <ssh-port>/tcp
ufw enable
```
**(Always audit and test scripts before using on production!)**

## References

- [Debian Security Documentation](https://wiki.debian.org/Hardening)
- [CIS Debian Benchmark](https://www.cisecurity.org/benchmark/debian_linux/)
- [Linux Security Guide](https://madaidans-insecurities.github.io/guides/linux-hardening.html)

**Harden every server as if it’s on the public internet — trust nothing, script everything, and review configs regularly.**

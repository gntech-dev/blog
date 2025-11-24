---
title: "Change Hostname on Linux (Ubuntu, Debian, CentOS, Fedora, Arch)"
date: 2025-11-24
categories: [linux]
tags: [hostname, sysadmin, linux]
---

This post shows safe, distro-agnostic ways to change the system hostname and covers common variations (systemd, init-based, NetworkManager, cloud-init, and containers).

**Quick summary**
- Temporary (current session): `sudo hostname <new>`
- Persistent (systemd systems): `sudo hostnamectl set-hostname <new>`
- Also update `/etc/hosts` and check cloud-init if present.

**Important:** Always verify whether your environment (cloud image, container, or managed host) uses cloud-init or another management tool — it may overwrite manual changes.

## 1 — systemd-based systems (Ubuntu, Debian, Fedora, Arch, CentOS 7+)

1. Set the persistent hostname:

   sudo hostnamectl set-hostname new-hostname

2. Verify:

   hostnamectl status
   hostname

3. Update `/etc/hosts` to map the new hostname to localhost or the server IP to avoid issues with services that resolve the hostname:

   sudo vim /etc/hosts

Example `/etc/hosts` entries:

```
127.0.0.1   localhost
127.0.1.1   new-hostname    # Debian/Ubuntu local host mapping
# or use the server's IP:
192.0.2.10  new-hostname.example.com new-hostname
```

4. If some services rely on the old name, either restart them or reboot:

   sudo systemctl restart systemd-hostnamed
   # or
   sudo reboot

## 2 — Older / init-style systems (non-systemd)

1. Change the running hostname (temporary):

   sudo hostname new-hostname

2. Make it persistent by writing to `/etc/hostname` (Debian-style):

   echo "new-hostname" | sudo tee /etc/hostname
   sudo vi /etc/hosts   # update hosts file as above

3. Reboot to apply fully, or run init/system-specific commands if available.

## 3 — NetworkManager-managed hosts

If NetworkManager manages the system hostname, use `nmcli`:

   sudo nmcli general hostname new-hostname

Then verify with `hostnamectl` or `hostname`.

## 4 — cloud-init managed instances

Cloud images often use cloud-init which can overwrite manual hostname changes on next boot or reconfiguration.

- To prevent cloud-init from overwriting your hostname, edit `/etc/cloud/cloud.cfg` and set:

  ```yaml
  preserve_hostname: true
  ```

- Or use cloud-init to set the hostname permanently (cloud-init configs vary by provider).

After changing cloud-init settings, you may need to re-run cloud-init or reboot.

## 5 — Containers and ephemeral environments

In containers (Docker, LXC), changing the hostname may be ephemeral and controlled by the container runtime. For Docker:

- At container creation: `docker run --hostname new-hostname ...`
- Inside a running container: `hostname new-hostname` (temporary)

To make changes persistent, configure the container runtime or the image startup scripts.

## 6 — Tips and verification

- Display the current hostname: `hostname`
- Show detailed status: `hostnamectl status`
- Fully-qualified domain name: `hostname -f` (ensure `/etc/hosts` contains the FQDN)
- Check `cat /etc/hostname` (on systems that use it)

## 7 — Troubleshooting

- Hostname reverts after reboot: check cloud-init, orchestration tools (Ansible, Salt, Puppet), or provider metadata.
- Services fail to start after renaming: ensure `/etc/hosts` maps the hostname correctly, and restart the affected services.
- NetworkManager or DHCP resets hostname: consider configuring the client or server to stop pushing hostnames, or set the hostname via the appropriate network tool.

---

If you want, I can:

- Add examples for a specific distro/version (e.g., RHEL 6, SUSE).
- Add screenshots or a short script to automate the change safely.

Want me to commit this post and add a short excerpt for the homepage?
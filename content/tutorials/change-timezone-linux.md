---
title: "How to Change Timezone in Linux (Ubuntu, CentOS, Fedora, Arch)"
date: 2025-11-24
categories: [linux]
tags: [timezone, sysadmin, linux]
---

Changing the timezone on a Linux system is a common task for system administrators, especially when managing servers in different regions or for users traveling. This guide covers how to change the timezone on various Linux distributions, including systemd-based systems and older ones.

## Prerequisites

- Root or sudo access to the system.
- Basic knowledge of the command line.

## 1. Check Current Timezone

Before changing the timezone, verify the current setting:

```bash
timedatectl  # For systemd systems
date          # General command
```

## 2. List Available Timezones

To see all available timezones:

```bash
timedatectl list-timezones  # For systemd systems
```

Or find your timezone:

```bash
timedatectl list-timezones | grep -i america  # Example for American timezones
```

## 3. Change Timezone on systemd-based Systems (Ubuntu 16.04+, CentOS 7+, Fedora, Arch)

Use `timedatectl` to set the timezone:

```bash
sudo timedatectl set-timezone America/New_York
```

Replace `America/New_York` with your desired timezone (e.g., `Europe/London`, `Asia/Tokyo`).

Verify the change:

```bash
timedatectl
```

## 4. Change Timezone on Older Systems (Ubuntu 14.04, CentOS 6, etc.)

For systems without systemd:

1. Install `tzdata` if not present:

   ```bash
   sudo apt install tzdata  # Ubuntu/Debian
   sudo yum install tzdata  # CentOS/RHEL
   ```

2. Run the timezone configuration tool:

   ```bash
   sudo dpkg-reconfigure tzdata  # Ubuntu/Debian
   sudo tzselect                    # Interactive selection
   ```

   Or manually set the timezone:

   ```bash
   sudo ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime
   ```

3. Update `/etc/timezone` if it exists:

   ```bash
   echo "America/New_York" | sudo tee /etc/timezone
   ```

## 5. Verify the Change

After changing the timezone, confirm it's set correctly:

```bash
date
```

The output should show the new timezone.

## 6. Troubleshooting

- **Timezone not persisting after reboot**: Ensure you're using the correct method for your system. On systemd systems, `timedatectl` is persistent.
- **NTP synchronization**: If using NTP, the time might sync automatically. Check with `timedatectl` or `systemctl status systemd-timesyncd`.
- **Container environments**: In Docker containers, timezone changes might not persist. Set it at runtime with `-e TZ=America/New_York`.

## Conclusion

Changing the timezone in Linux is straightforward once you know the right commands for your distribution. For modern systems, `timedatectl` is the recommended tool. Always verify the change and consider the impact on scheduled tasks or logs.

For more Linux sysadmin tips, check out our other tutorials!
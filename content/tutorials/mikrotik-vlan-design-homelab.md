---
title: "Designing a VLAN-Based Homelab Network with MikroTik"
date: 2026-03-15
categories: ["Networking", "MikroTik", "Homelab"]
tags: ["mikrotik", "vlan", "homelab", "network-design", "routeros", "ipv6"]
author: "gnolasco"
draft: false
---

Segmenting a home network using VLANs brings many of the benefits of enterprise network design into a homelab environment.

In this article I explain how I designed a **secure and scalable VLAN architecture using a MikroTik router**.

The goal was to:

- isolate IoT devices
- protect management infrastructure
- support development environments
- enable camera networks
- maintain simple firewall policies

## Why Use VLANs in a Homelab

Many home networks place everything on a single subnet.

That means:

- IoT devices can see everything
- cameras can scan your network
- development systems share broadcast domains
- troubleshooting becomes harder

VLAN segmentation solves these issues.

Benefits include:

- **security isolation**
- **simpler firewall rules**
- **clean network organization**
- **scalable architecture**

## Network Overview

The MikroTik router acts as the **Layer 3 gateway** for all networks.

```text
                        Internet
                           │
                        PPPoE
                           │
                      MikroTik Router
                           │
                     bridge-trunk
                  (VLAN filtering enabled)
                           │
          ┌───────────────┼────────────────┐
          │               │                │
       Trunk           Access           Access
       ether5          ether2           ether4
     (Switch)          HOME           Dumb AP
```

Wireless networks are mapped to VLANs as well.

## VLAN Architecture

Each network has a dedicated VLAN and subnet.

| VLAN | Subnet | Purpose |
|-----|--------|--------|
| 10 | 10.0.10.0/24 | Home devices |
| 20 | 10.0.20.0/24 | Development lab |
| 30 | 10.0.30.0/24 | Production services |
| 40 | 10.0.40.0/24 | IoT devices |
| 50 | 10.0.50.0/24 | CCTV cameras |
| 99 | 10.0.99.0/24 | Network management |

Each VLAN has its own gateway:

```text
10.0.10.1 → HOME
10.0.20.1 → LAB
10.0.30.1 → PRODUCTION
10.0.40.1 → IoT
10.0.50.1 → CCTV
10.0.99.1 → MANAGEMENT
```

## Wireless Network Mapping

Wireless networks map directly to VLANs.

| SSID | VLAN | Purpose |
|-----|------|-------|
| DEV | 20 | Development network |
| GNTECH | 30 | Production services |
| IoT | 40 | Smart devices |
| GS | 50 | CCTV cameras |

This allows wireless clients to be segmented exactly like wired clients.

## Security Model

The firewall follows a **default deny model**.

### Router Protection

Only management VLAN devices can access the router.

```text
MGMT VLAN → Router
All other VLANs → blocked
WAN → blocked
```

### Inter-VLAN Policy

Most VLANs cannot communicate with each other.

```text
HOME → Internet
LAB → Internet
GNTECH → Internet
IoT → Internet only
CCTV → Internet only
MGMT → Full access
```

This prevents compromised IoT or camera devices from reaching the rest of the network.

## Service Exceptions

Certain services require controlled cross-VLAN access.

Example:

```text
Home Assistant → Camera network
```

```text
10.0.20.10 → 10.0.50.0/24
```

This allows Frigate or Home Assistant to access cameras while keeping the rest of the networks isolated.

## DNS Design

Each VLAN uses the router as its DNS server.

Example:

```text
10.0.10.1
10.0.20.1
10.0.30.1
10.0.40.1
10.0.50.1
10.0.99.1
```

The router forwards requests to upstream DNS providers.

```text
1.1.1.1
8.8.8.8
```

Advantages:

- local caching
- easier firewall policies
- simplified DNS management

## IPv6 Integration

The router requests an IPv6 prefix from the ISP.

Each VLAN automatically receives an IPv6 subnet using prefix delegation.

```text
ISP IPv6 /56
        │
   RouterOS PD
        │
   /64 per VLAN
```

Benefits:

- full IPv6 connectivity
- SLAAC client addressing
- future-proof network design

## Homelab Services

This network design supports multiple services:

- Home Assistant
- Frigate NVR
- container workloads
- development environments
- testing environments

Segmentation ensures that experimental workloads do not affect the rest of the network.

## Hardware Used

Router:

```text
MikroTik hAP ac²
RBD52G-5HacD2HnD
RouterOS 7
```

WAN:

```text
PPPoE ISP connection
IPv6 prefix delegation
```

## Lessons Learned

Several design decisions made the network easier to manage.

### Use a single bridge with VLAN filtering

This simplifies trunk management.

### Always implement a default deny firewall

Explicit allow rules prevent accidental exposure.

### Separate management networks

Administrative interfaces should never be reachable from IoT or guest devices.

### Plan VLANs early

Expanding VLANs later can be disruptive.

## Future Improvements

Possible improvements include:

- WireGuard remote access
- centralized logging
- network monitoring
- Prometheus metrics
- NetFlow analysis
- multi-site connectivity

## Final Thoughts

A properly segmented homelab network dramatically improves both **security and maintainability**.

With VLANs, firewall policies, and IPv6 support, a MikroTik router can deliver a powerful and flexible architecture suitable for advanced home labs.

If you're building your own homelab network, start with segmentation first. Everything else becomes easier once the architecture is well defined.

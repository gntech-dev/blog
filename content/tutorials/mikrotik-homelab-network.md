---
title: "Building a Secure MikroTik Homelab Network with VLANs, IPv6, and PPPoE"
date: 2026-03-15
categories: ["Networking", "MikroTik", "Homelab"]
tags: ["mikrotik", "routeros", "vlan", "ipv6", "homelab", "networking"]
author: "gnolasco"
draft: false
---

Modern homelabs often run multiple services: development environments, smart home systems, cameras, containers, and infrastructure experiments. Without proper segmentation, these services all share the same broadcast domain and security boundary.

To improve both **security and manageability**, I redesigned my home network using **VLAN segmentation, strict firewall rules, IPv6 support, and PPPoE WAN connectivity** on a MikroTik router.

This article walks through the design decisions and configuration used to build a **secure, scalable MikroTik homelab network**.

## Hardware and Software

Router:

- **MikroTik hAP ac² (RBD52G-5HacD2HnD)**
- **RouterOS 7.20**
- ISP connection via **PPPoE**

The router acts as the **Layer-3 gateway** for all networks and enforces segmentation between them.

## Network Architecture

The network uses a **single bridge with VLAN filtering** enabled. All VLANs are trunked internally and selectively exposed to ports and wireless networks.

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
         ┌─────────────┼─────────────┐
         │             │             │
      ether2        ether4        ether5
      HOME         Dumb AP        Trunk
                                     │
                                 Switch
```

The router performs:

- routing between VLANs
- firewall enforcement
- DHCP services
- DNS forwarding
- IPv6 prefix delegation

## VLAN Design

Segmenting the network into logical groups simplifies security policies and prevents unnecessary communication between devices.

| VLAN | Subnet | Purpose |
|-----|--------|--------|
| 10 | 10.0.10.0/24 | Home network |
| 20 | 10.0.20.0/24 | Development / lab |
| 30 | 10.0.30.0/24 | Production services |
| 40 | 10.0.40.0/24 | IoT devices |
| 50 | 10.0.50.0/24 | CCTV cameras |
| 99 | 10.0.99.0/24 | Management network |

Each VLAN has a dedicated router interface acting as the gateway.

Example:

```text
10.0.10.1 → Home network
10.0.20.1 → Development network
10.0.30.1 → Production network
10.0.40.1 → IoT network
10.0.50.1 → CCTV network
10.0.99.1 → Management network
```

This segmentation allows strict control over how devices interact with one another.

## Wireless Network Segmentation

Wireless networks map directly to VLANs, allowing the same segmentation for Wi-Fi clients.

| SSID | VLAN | Purpose |
|-----|------|------|
| DEV | 20 | Development |
| GNTECH | 30 | Production |
| IoT | 40 | Smart devices |
| GS | 50 | Cameras |

This ensures that a wireless device connected to the IoT SSID cannot communicate with the development or management networks.

## DHCP and DNS Design

Each VLAN runs its own DHCP scope.

Example configuration:

```text
10.0.10.0/24 → gateway 10.0.10.1
10.0.20.0/24 → gateway 10.0.20.1
10.0.30.0/24 → gateway 10.0.30.1
```

Clients use the router interface as their DNS server. The router then forwards queries to upstream resolvers:

```text
1.1.1.1
8.8.8.8
```

Advantages:

- DNS caching
- centralized DNS policy
- easier firewall rules
- reduced external DNS traffic

## Firewall Design

The firewall follows a **default-deny model**.

All traffic is blocked unless explicitly allowed.

### Protecting the Router

The INPUT chain controls access to the router itself.

Rules allow:

- established connections
- DHCP
- DNS queries
- ICMP
- management access from VLAN 99

Everything else is dropped.

Example logic:

```text
accept established,related
drop invalid
accept ICMP
accept DHCP
accept DNS
accept MGMT VLAN
drop WAN access
drop all remaining traffic
```

## Inter-VLAN Policies

Most VLANs are isolated from each other.

Allowed flows:

| Source | Destination | Purpose |
|------|------|------|
| HOME | Internet | normal browsing |
| LAB | Internet | development traffic |
| GNTECH | Internet | services |
| IoT | Internet | smart devices |
| CCTV | Internet | firmware updates |
| MGMT | All VLANs | administration |

Everything else is blocked.

## Service Exceptions

Some services require controlled cross-VLAN communication.

For example, **Home Assistant / Frigate** must access cameras.

```text
Source: 10.0.20.10
Destination: 10.0.50.0/24
```

This allows the NVR to connect to cameras without exposing the entire CCTV network.

## IPv6 Configuration

The router requests a delegated IPv6 prefix from the ISP using DHCPv6-PD.

```bash
/ipv6 dhcp-client
add interface=pppoe-out1 request=prefix pool-name=ipv6-pd
```

Each VLAN receives a /64 prefix from that pool.

```bash
/ipv6 address
add from-pool=ipv6-pd interface=vlan-home
add from-pool=ipv6-pd interface=vlan-lab
add from-pool=ipv6-pd interface=vlan-gntech
```

This enables full IPv6 connectivity while maintaining VLAN separation.

## NAT Configuration

Internet access for IPv4 is provided via a simple masquerade rule.

```bash
/ip firewall nat
add chain=srcnat action=masquerade out-interface=pppoe-out1
```

## Security Benefits

This design provides several advantages:

- isolation of insecure IoT devices
- protection of infrastructure systems
- simplified firewall policies
- scalable network layout
- safer experimentation for homelab environments

If a device becomes compromised, its impact is limited to its VLAN.

## Lessons Learned

A few best practices proved particularly useful:

**Use a single VLAN-aware bridge**

RouterOS VLAN filtering simplifies trunk management and avoids unnecessary complexity.

**Adopt a default-deny firewall**

Explicit allow rules prevent accidental exposure.

**Separate management traffic**

Administrative interfaces should always reside on a dedicated management network.

**Plan segmentation early**

Redesigning VLAN layouts later can be disruptive.

## Future Improvements

Possible improvements for this network include:

- WireGuard remote access
- centralized logging
- Prometheus monitoring
- network flow analysis
- multi-site connectivity

## Conclusion

With VLAN segmentation, strict firewall policies, and IPv6 support, a MikroTik router can deliver a powerful enterprise-style architecture even in a home lab.

A well-structured network not only improves security but also makes it easier to expand the lab as new services and infrastructure are added.

The key takeaway is simple:

**Design the network architecture first — everything else becomes easier afterward.**

---
title: "MikroTik VLAN Homelab Configuration Reference"
date: 2026-03-15
categories: ["Networking", "MikroTik", "Homelab"]
tags: ["mikrotik", "routeros", "vlan", "config", "ipv6", "firewall"]
author: "gnolasco"
draft: false
---

This page contains a sanitized reference configuration for the MikroTik homelab network described in the main article. Sensitive values such as PPPoE credentials have been replaced with placeholders.

## PPPoE WAN Configuration

Replace the placeholders below with your ISP credentials.

```bash
/interface pppoe-client
add name=pppoe-out1     interface=ether1     user="ISP_USERNAME"     password="ISP_PASSWORD"     add-default-route=yes     use-peer-dns=no
```

## Bridge Configuration

```bash
/interface bridge
add name=bridge-trunk vlan-filtering=yes comment="Main trunk bridge for all VLANs"
```

## VLAN Interfaces

```bash
/interface vlan
add name=vlan-home vlan-id=10 interface=bridge-trunk comment="HOME"
add name=vlan-lab vlan-id=20 interface=bridge-trunk comment="LAB"
add name=vlan-gntech vlan-id=30 interface=bridge-trunk comment="PRODUCTION"
add name=vlan-iot vlan-id=40 interface=bridge-trunk comment="IoT"
add name=vlan-cctv vlan-id=50 interface=bridge-trunk comment="CCTV"
add name=vlan-mgmt vlan-id=99 interface=bridge-trunk comment="MANAGEMENT"
```

## Bridge Port Configuration

```bash
/interface bridge port

# HOME access port
add bridge=bridge-trunk interface=ether2 pvid=10 frame-types=admit-only-untagged-and-priority-tagged

# Dumb AP access port
add bridge=bridge-trunk interface=ether4 pvid=10 frame-types=admit-only-untagged-and-priority-tagged

# VLAN trunk
add bridge=bridge-trunk interface=ether5 frame-types=admit-only-vlan-tagged

# Wireless mappings
add bridge=bridge-trunk interface=wlan2 pvid=20
add bridge=bridge-trunk interface=wlan2-gntech pvid=30
add bridge=bridge-trunk interface=wlan1-iot pvid=40
add bridge=bridge-trunk interface=wlan1 pvid=50
```

## Bridge VLAN Table

```bash
/interface bridge vlan
add bridge=bridge-trunk vlan-ids=10 tagged=bridge-trunk,ether5 untagged=ether2,ether4
add bridge=bridge-trunk vlan-ids=20 tagged=bridge-trunk,ether5 untagged=wlan2
add bridge=bridge-trunk vlan-ids=30 tagged=bridge-trunk,ether5 untagged=wlan2-gntech
add bridge=bridge-trunk vlan-ids=40 tagged=bridge-trunk,ether5 untagged=wlan1-iot
add bridge=bridge-trunk vlan-ids=50 tagged=bridge-trunk,ether5 untagged=wlan1
add bridge=bridge-trunk vlan-ids=99 tagged=bridge-trunk,ether5
```

## IP Addressing

```bash
/ip address
add address=10.0.10.1/24 interface=vlan-home comment="HOME Gateway"
add address=10.0.20.1/24 interface=vlan-lab comment="LAB Gateway"
add address=10.0.30.1/24 interface=vlan-gntech comment="PROD Gateway"
add address=10.0.40.1/24 interface=vlan-iot comment="IoT Gateway"
add address=10.0.50.1/24 interface=vlan-cctv comment="CCTV Gateway"
add address=10.0.99.1/24 interface=vlan-mgmt comment="MGMT Gateway"
```

## DHCP Pools

```bash
/ip pool
add name=pool-home ranges=10.0.10.100-10.0.10.250
add name=pool-lab ranges=10.0.20.100-10.0.20.250
add name=pool-gntech ranges=10.0.30.100-10.0.30.250
add name=pool-iot ranges=10.0.40.100-10.0.40.250
add name=pool-cctv ranges=10.0.50.100-10.0.50.250
add name=pool-mgmt ranges=10.0.99.100-10.0.99.250
```

## DHCP Servers

```bash
/ip dhcp-server
add name=dhcp-home interface=vlan-home address-pool=pool-home
add name=dhcp-lab interface=vlan-lab address-pool=pool-lab disabled=yes
add name=dhcp-gntech interface=vlan-gntech address-pool=pool-gntech
add name=dhcp-iot interface=vlan-iot address-pool=pool-iot
add name=dhcp-cctv interface=vlan-cctv address-pool=pool-cctv
add name=dhcp-mgmt interface=vlan-mgmt address-pool=pool-mgmt
```

## DHCP Networks

```bash
/ip dhcp-server network
add address=10.0.10.0/24 gateway=10.0.10.1 dns-server=10.0.10.1
add address=10.0.20.0/24 gateway=10.0.20.1 dns-server=10.0.20.1
add address=10.0.30.0/24 gateway=10.0.30.1 dns-server=10.0.30.1
add address=10.0.40.0/24 gateway=10.0.40.1 dns-server=10.0.40.1
add address=10.0.50.0/24 gateway=10.0.50.1 dns-server=10.0.50.1
add address=10.0.99.0/24 gateway=10.0.99.1 dns-server=10.0.99.1
```

## DNS Forwarding

```bash
/ip dns
set allow-remote-requests=yes servers=1.1.1.1,8.8.8.8
```

## Firewall (IPv4)

```bash
/ip firewall filter
add chain=input action=accept connection-state=established,related comment="V2 INPUT established,related"
add chain=input action=drop connection-state=invalid comment="V2 INPUT drop invalid"
add chain=input action=accept protocol=icmp limit=5,5:packet comment="V2 INPUT ICMP"
add chain=input action=accept protocol=udp dst-port=67 in-interface-list=LAN comment="V2 INPUT DHCP from LAN"
add chain=input action=accept protocol=udp dst-port=53 in-interface-list=LAN comment="V2 INPUT DNS UDP from LAN"
add chain=input action=accept protocol=tcp dst-port=53 in-interface-list=LAN comment="V2 INPUT DNS TCP from LAN"
add chain=input action=accept in-interface=vlan-mgmt comment="V2 INPUT MGMT to router"
add chain=input action=drop in-interface-list=WAN comment="V2 INPUT drop WAN"
add chain=input action=drop comment="V2 INPUT drop all"

add chain=forward action=fasttrack-connection connection-state=established,related hw-offload=yes comment="V2 FWD fasttrack"
add chain=forward action=accept connection-state=established,related comment="V2 FWD established,related"
add chain=forward action=drop connection-state=invalid comment="V2 FWD drop invalid"
add chain=forward action=accept in-interface=vlan-mgmt out-interface-list=LAN comment="V2 FWD MGMT to all VLANs"
add chain=forward action=accept connection-state=new in-interface=vlan-home out-interface=pppoe-out1 comment="V2 FWD HOME to internet"
add chain=forward action=accept connection-state=new in-interface=vlan-lab out-interface=pppoe-out1 comment="V2 FWD LAB to internet"
add chain=forward action=accept connection-state=new in-interface=vlan-gntech out-interface=pppoe-out1 comment="V2 FWD GNTECH to internet"
add chain=forward action=accept connection-state=new in-interface=vlan-iot out-interface=pppoe-out1 comment="V2 FWD IoT to internet"
add chain=forward action=accept connection-state=new in-interface=vlan-cctv out-interface=pppoe-out1 comment="V2 FWD CCTV to internet"
add chain=forward action=accept connection-state=new in-interface=vlan-mgmt out-interface=pppoe-out1 comment="V2 FWD MGMT to internet"
add chain=forward action=accept src-address=10.0.10.0/24 dst-address=10.0.20.10 comment="V2 FWD HOME to HA/Frigate"
add chain=forward action=accept src-address=10.0.20.10 dst-address=10.0.50.0/24 comment="V2 FWD HA/Frigate to CCTV"
add chain=forward action=accept protocol=udp dst-address=224.0.0.251 dst-port=5353 in-interface=vlan-home out-interface=vlan-iot comment="V2 FWD mDNS Home to IoT"
add chain=forward action=accept protocol=udp dst-address=224.0.0.251 dst-port=5353 in-interface=vlan-iot out-interface=vlan-home comment="V2 FWD mDNS IoT to Home"
add chain=forward action=accept protocol=udp dst-port=1900 in-interface=vlan-home out-interface=vlan-iot comment="V2 FWD SSDP Home to IoT"
add chain=forward action=accept protocol=udp dst-port=1900 in-interface=vlan-iot out-interface=vlan-home comment="V2 FWD SSDP IoT to Home"
add chain=forward action=drop comment="V2 FWD drop remaining inter-VLAN"
```

## NAT

```bash
/ip firewall nat
add chain=srcnat action=masquerade out-interface=pppoe-out1 comment="NAT internet access via PPPoE"
```

## IPv6

```bash
/ipv6 dhcp-client
add interface=pppoe-out1 request=prefix pool-name=ipv6-pd add-default-route=yes
```

Assign delegated prefixes to VLANs:

```bash
/ipv6 address
add from-pool=ipv6-pd interface=vlan-home
add from-pool=ipv6-pd interface=vlan-lab
add from-pool=ipv6-pd interface=vlan-gntech
add from-pool=ipv6-pd interface=vlan-iot
add from-pool=ipv6-pd interface=vlan-cctv
add from-pool=ipv6-pd interface=vlan-mgmt
```

Neighbor discovery per VLAN:

```bash
/ipv6 nd
set [find default=yes] disabled=yes
add interface=vlan-home
add interface=vlan-lab
add interface=vlan-gntech
add interface=vlan-iot
add interface=vlan-cctv
add interface=vlan-mgmt
```

## Firewall (IPv6)

```bash
/ipv6 firewall filter
add chain=input action=accept connection-state=established,related comment="V2 IPv6 INPUT established,related"
add chain=input action=drop connection-state=invalid comment="V2 IPv6 INPUT drop invalid"
add chain=input action=accept protocol=icmpv6 comment="V2 IPv6 INPUT ICMPv6"
add chain=input action=accept protocol=udp dst-port=546 comment="V2 IPv6 INPUT DHCPv6 client"
add chain=input action=accept protocol=udp dst-port=53 in-interface-list=LAN comment="V2 IPv6 INPUT DNS UDP from LAN"
add chain=input action=accept protocol=tcp dst-port=53 in-interface-list=LAN comment="V2 IPv6 INPUT DNS TCP from LAN"
add chain=input action=accept in-interface=vlan-mgmt comment="V2 IPv6 INPUT MGMT to router"
add chain=input action=drop in-interface-list=WAN comment="V2 IPv6 INPUT drop WAN"
add chain=input action=drop comment="V2 IPv6 INPUT drop all"

add chain=forward action=accept connection-state=established,related comment="V2 IPv6 FWD established,related"
add chain=forward action=drop connection-state=invalid comment="V2 IPv6 FWD drop invalid"
add chain=forward action=accept protocol=icmpv6 comment="V2 IPv6 FWD ICMPv6"
add chain=forward action=accept in-interface=vlan-mgmt out-interface-list=LAN comment="V2 IPv6 FWD MGMT to all VLANs"
add chain=forward action=accept connection-state=new in-interface=vlan-home out-interface=pppoe-out1 comment="V2 IPv6 FWD HOME to internet"
add chain=forward action=accept connection-state=new in-interface=vlan-lab out-interface=pppoe-out1 comment="V2 IPv6 FWD LAB to internet"
add chain=forward action=accept connection-state=new in-interface=vlan-gntech out-interface=pppoe-out1 comment="V2 IPv6 FWD GNTECH to internet"
add chain=forward action=accept connection-state=new in-interface=vlan-iot out-interface=pppoe-out1 comment="V2 IPv6 FWD IoT to internet"
add chain=forward action=accept connection-state=new in-interface=vlan-cctv out-interface=pppoe-out1 comment="V2 IPv6 FWD CCTV to internet"
add chain=forward action=accept connection-state=new in-interface=vlan-mgmt out-interface=pppoe-out1 comment="V2 IPv6 FWD MGMT to internet"
add chain=forward action=drop comment="V2 IPv6 FWD drop remaining inter-VLAN"
```

## Notes

This reference configuration provides:

- VLAN segmentation
- PPPoE WAN
- IPv4 NAT
- IPv6 prefix delegation
- per-VLAN DHCP and DNS
- management network isolation
- IoT and CCTV containment

Adapt IP ranges, SSIDs, and PPPoE credentials to match your own environment.

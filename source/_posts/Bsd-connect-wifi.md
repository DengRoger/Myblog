---
title: Bsd connect wifi
date: 2023-05-13 07:36:25
categories:
  - System
tags:
  - NA
  - bsd
  - wifi
---


# FreeBsd linked and disconnect network
<!-- more -->

1. linked default route and setup dns 
* create route 

```
    ifconfig <network interface> <self_ip> netmask <255.255.255.0> 
    dhclient <network interface> 
```

* delete route 
```
    route del <ip> 
```
* setup default gateway 
```
    route add default <default_gateway>
```
2. linked wifi 
```
    use ifconfig to see if NetworkInterface is up 
    ifconfig <NetworkInterface> up 
    install net-tools first : apt install net-tools 
    wpa_passphrase [SSID_NAME] >> /etc/wpa_supplicant.conf
    wpa_supplicant -B -D <drive> -i <NetworkInterface> -c /etc/wpa_supplicant.conf
    //drive = nl80211  
    dhclient <NetworkInterface> 
```
3. setup static inet 
    * at /etc/rc.conf setup:
```
    hostname="freebsd12"
    ifconfig_em0="DHCP"
    ifconfig_em0_ipv6="inet6 accept_rtadv"
    sshd_enable="YES"
    # Set dumpdev to "AUTO" to enable crash dumps, "NO" to disable
    dumpdev="AUTO"
```
# Secure CCNA Lab Integration

## Project Overview

This project establishes a **physical CCNA practice lab** designed to isolate experimental network configurations from the primary home network while still allowing controlled internet access.

A consumer router (**Linksys E2500**) is used as a **safety buffer** between the upstream home network and the Cisco lab environment. This ensures that configuration errors, routing changes, or future experiments do not disrupt the main household network.

The **Cisco 2600 router** functions as the **lab gateway**, initially receiving its upstream address via DHCP from the buffer router. The interface was later converted to a static configuration to ensure stable management addressing.

During remote management configuration it was discovered that the router's IOS image does not support crypto features required for SSH. As a result, Telnet-based remote access was implemented while planning an IOS upgrade to enable SSH in a future phase.
This lab environment will continue to evolve as additional devices (switches and endpoints) are integrated.

---

# Lab Goals

The objectives of this lab setup are:

* Create a **safe and isolated networking environment** for CCNA experimentation
* Prevent lab configurations from impacting the **primary home network**
* Establish **internet connectivity** for the lab environment
* Implement **remote management access to the router**
* Lay the groundwork for a **multi-device Cisco lab topology**

---

# Hardware Inventory

The following hardware is currently used or planned for the lab.

| Device                 | Role                                              |
| ---------------------- | ------------------------------------------------- |
| Linksys E2500          | Safety buffer router between home network and lab |
| Cisco 2600 Router      | Lab gateway router                                |
| Cisco 1700 Router      | Additional router for future lab scenarios        |
| Catalyst 3500XL Switch | Planned core lab LAN switch                       |
| MacBook                | Control station for console management            |
| Raspberry Pi           | Planned lab endpoint / TFTP server                |

---

# Network Topology

The topology below illustrates the **current lab architecture and planned expansion**.

## Current Network Flow

```
Internet
   ↓
Xfinity Gateway
   ↓
Linksys E2500 (NAT + DHCP)
   ↓
Cisco 2600 Router (Static WAN Interface)
   ↓
Console Management from MacBook
```

## Planned Expansion

```
Cisco 2600
   ↓
Catalyst 3500XL Switch
   ↓
Internal Lab LAN
   ↓
Lab Endpoints (Raspberry Pi, additional hosts)
```

---

# Physical Connections

The lab hardware is connected as follows:

| Connection                                 | Description                                       |
| ------------------------------------------ | ------------------------------------------------- |
| Xfinity Gateway → Linksys WAN              | Connects the lab network to the upstream internet |
| Linksys LAN Port 1 → Cisco FastEthernet0/1 | Provides DHCP address to Cisco router             |
| MacBook USB → Cisco Console Port           | Initial router configuration via terminal         |

This structure places the Cisco router **behind the Linksys buffer**, ensuring the lab remains isolated.

---

# Buffer Router Configuration (Linksys E2500)

The Linksys router acts as a **safety buffer** between the home network and the CCNA lab.

This router performs:

* NAT for internet access
* DHCP address assignment for the lab gateway
* Wireless access control

---

## Local Network Configuration

The default LAN address was changed to prevent conflicts with the upstream home network.

```
Router LAN IP: 192.168.50.1
Subnet: 192.168.50.0/24
DHCP Range: 192.168.50.x
```

This ensures the lab network does not overlap with the upstream **192.168.1.x** home network.

---

## Wireless Configuration

Wireless settings were hardened to prevent unauthorized access.

```
SSID: CCNA_Lab
Encryption: WPA2 Personal (AES)
```

The router was switched from **WPS mode to manual configuration** to allow secure SSID customization.

---

## Management Security

The default router administration password was changed under:

```
Administration → Management
```

This prevents unauthorized modification of the buffer router.

---

# Cisco 2600 Router Configuration

The Cisco 2600 router serves as the **gateway device for the CCNA lab environment**.

It currently performs the following functions:

* Static WAN connectivity to the buffer router
* Traffic routing for the future lab network
* Remote management via Telnet (VTY lines)
  
---

## WAN Interface Configuration

The router was configured to obtain its upstream address from the Linksys router.

```
interface FastEthernet0/1
 ip address dhcp
 no shutdown
```

After configuration, the router received the following address:

```
Assigned IP: 192.168.50.131
Because DHCP assignments can change between reboots, the interface was converted to a static configuration to ensure stable management access.

Updated configuration:

Router# configure terminal
Router(config)# interface FastEthernet0/1
Router(config-if)# no ip address dhcp
Router(config-if)# ip address 192.168.50.10 255.255.255.0
Router(config-if)# no shutdown
```

---

## SSH Preparation (In Progress)

The router configuration was prepared for future SSH access.

```
ip domain-name ccnahome.lab
```

This setting is required before generating RSA keys for SSH encryption.

Planned configuration:

```
crypto key generate rsa
```

SSH will allow the router to be managed remotely without the console cable.

---

# Verification and Testing

Connectivity tests were performed to confirm proper network operation.

---

## Gateway Connectivity Test

The Cisco router successfully reached the upstream buffer router.

```
ping 192.168.50.1
```

**Screenshot**

*(Add screenshot here)*

---

## Internet Connectivity Test

External connectivity was verified using Google's public DNS server.

```
ping 8.8.8.8
```

**Screenshot**

*(Add screenshot here)*

---

# Configuration Persistence

After verification, the configuration was saved to NVRAM.

```
write memory
```

This ensures the router retains its configuration after a reboot.

---

# Current Lab State

The lab environment currently provides:

* An **isolated CCNA practice environment**
* Static IP connectivity to the buffer router
* Default routing to the upstream gateway
* Telnet-based remote management via VTY lines
* Verified internet reachability
* Console-based management from the MacBook control station

The **internal LAN and switch infrastructure have not yet been deployed**.

---

# Next Steps / Planned Expansion

The next phase of the lab will expand the network topology.

---

## IOS Upgrade
The current router IOS image does not support the crypto features required for SSH.

IOS Version:
Cisco IOS Software (C2600-D-M), Version 12.1(3)T

To enable SSH remote management, the router IOS will be upgraded via TFTP to a **crypto-capable image**.

After upgrading, SSH access will be configured.

---

## SSH Remote Management

After the IOS upgrade, secure remote access will be configured using SSH.

Planned configuration:

```
Router# configure terminal
Router(config)# ip domain-name ccnahome.lab
Router(config)# username admin privilege 15 secret <password>
Router(config)# crypto key generate rsa
Router(config)# line vty 0 4
Router(config-line)# transport input ssh
Router(config-line)# login local
```

This configuration will allow secure remote management of the router without requiring console access.
---

## Switch Integration

The **Catalyst 3500XL switch** will be connected to create the internal lab LAN.

Planned network:

```
10.0.0.0/24
```

---

## Endpoint Integration

The **Raspberry Pi** will be added to the lab environment and may serve several roles:

* TFTP server for IOS backups
* Linux lab endpoint
* Automation and scripting platform

---

# Summary

This project establishes the foundation for a **safe and expandable CCNA practice lab** using physical Cisco networking equipment.

The environment isolates experimental configurations from the primary home network while still allowing controlled internet access and future expansion into a full multi-device network lab.

---

# Next Repository Updates

Future commits will include:

* SSH implementation
* Switch configuration
* Internal lab subnet deployment
* Additional endpoint integration


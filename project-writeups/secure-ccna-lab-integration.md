# Secure CCNA Lab Integration — Technical Writeup

**Repo:** [cornerstonian/netops-ccna-homelab](https://github.com/cornerstonian/netops-ccna-homelab)
**Status:** Active — Lab 04 in progress
**Last Updated:** April 2026

---

## Lab Goals

| Goal | Status |
|---|---|
| Isolate lab network from primary home network | ✅ Complete |
| Establish internet connectivity for lab environment | ✅ Complete |
| Implement remote management access | ✅ Complete — Telnet (2621), SSH in progress (1700) |
| Build multi-device Cisco lab topology | 🔄 In Progress |
| Integrate switch and internal LAN | 📋 Planned |
| Integrate Raspberry Pi endpoint | 📋 Planned |

---

## Hardware Inventory

| Device | Role | Status |
|---|---|---|
| Linksys E2500 | Buffer router — NAT, DHCP, lab isolation | ✅ Active |
| Cisco 2621 — NetOps-R1 | Lab gateway — Telnet / cleartext demo | ✅ Active |
| Cisco 1700 | SSH lab device — advsecurityk9, crypto-capable | ✅ Active |
| Catalyst 3500XL Switch | Core lab LAN switch | 📋 Planned |
| MacBook Pro 2015 | Console management / capture station | ✅ Active |
| Raspberry Pi | TFTP server / Linux endpoint / automation | 📋 Planned |
| Insignia USB-A Gigabit Ethernet | MacBook capture interface (en4) | ✅ Active |

---

## Network Topology

### Current

```
Internet
   ↓
Xfinity Gateway — 192.168.1.x (upstream home network)
   ↓
Linksys E2500 — 192.168.50.1
(Buffer Router — NAT + DHCP — isolation boundary)
   ↓
   ├── Cisco 2621 — NetOps-R1 — 192.168.50.10 (Fa0/1)
   │   IOS 12.1(3)T — base image, no crypto
   │   Remote access: Telnet (VTY 0-4)
   │
   └── Cisco 1700 — 192.168.50.x (TBD)
       IOS 12.4(25d) — advsecurityk9
       Remote access: SSH (Lab 04 — in progress)

MacBook Pro — 192.168.50.147 (en4)
   ├── Console → Cisco 2621 (USB serial)
   ├── Console → Cisco 1700 (USB serial — one at a time)
   └── Wireshark capture on en4
```

### Planned Expansion

```
Cisco 2621 — NetOps-R1
   ↓
Catalyst 3500XL Switch (internal subnet TBD)
   ↓
   ├── Raspberry Pi — TFTP server / Linux endpoint / automation
   └── Additional lab endpoints
```

---

## Physical Connections

| Connection | Detail |
|---|---|
| Xfinity Gateway → Linksys WAN | Upstream internet feed |
| Linksys LAN → Cisco 2621 Fa0/1 | Static IP 192.168.50.10 |
| Linksys LAN → Cisco 1700 | Planned — Lab 04 |
| MacBook USB → Console port | USB serial — `/dev/tty.usbserial-A9Q7R5CX` |
| Linksys DHCP → MacBook en4 | 192.168.50.147 |

---

## Buffer Router Configuration — Linksys E2500

### Purpose

Isolates the Cisco lab from the upstream home network. Lab misconfigurations, routing changes, and experimental configs cannot reach `192.168.1.x`.

### Network Configuration

| Setting | Value |
|---|---|
| Router Name | NetOpsZero |
| LAN IP | 192.168.50.1 |
| Subnet | 192.168.50.0/24 |
| DHCP Range | 192.168.50.100 — 192.168.50.149 |
| SSID | NetOpsLab |
| Encryption | WPA2 Personal (AES) |
| Admin password | Changed from default |

> Subnet changed from default `192.168.1.x` to avoid overlap with upstream home network.
> WPS disabled. Manual SSID configuration applied.
> Admin password changed under Administration → Management.

![Linksys E2500 buffer router admin configuration](../images/linksys-e2500-buffer-config-gui.jpg)

---

## Cisco 2621 Configuration — NetOps-R1

### Device Info

| Setting | Value |
|---|---|
| IOS Image | `c2600-d-mz.121-3.T` |
| IOS Version | 12.1(3)T |
| Crypto Support | None — base image |
| Active Interface | FastEthernet0/1 |
| IP Address | 192.168.50.10 / 255.255.255.0 |
| Remote Access | Telnet — VTY 0-4 |
| Enable Password | Set |
| VTY Password | Set |
| Config Register | 0x2102 |
| Flash | 16MB total — 11.4MB available |

### WAN Interface Configuration

Initial config attempted DHCP on Fa0/0 — produced a Bus Error exception and router crash. See [Bus Error — Troubleshooting](#bus-error-exception--fa00-dhcp) below.

Corrected config used Fa0/1 with DHCP first:

```
interface FastEthernet0/1
 ip address dhcp
 no shutdown
```

DHCP assigned: `192.168.50.131`

Converted to static for stable management addressing:

```
interface FastEthernet0/1
 no ip address dhcp
 ip address 192.168.50.10 255.255.255.0
 no shutdown
```

![Fa0/1 DHCP assignment, ping verification, write memory, internet reachability](../images/cisco-2600-verification-ping-test.jpg)

> **Interface note:** Physical uplink cable runs to Fa0/1. Assigning IP to Fa0/0 produces `up/down` — no physical link on that port. IP must be on Fa0/1.

### VTY Configuration

```
line vty 0 4
 password cisco
 login
 transport input telnet
```

### Verification

```
ping 192.168.50.1    # Linksys — 5/5 success
ping 8.8.8.8         # Internet — 5/5 success
write memory         # Configuration persisted to NVRAM
```

> `ttl=255` on ping reply confirms Cisco IOS. Linux-based devices return `ttl=64`.

---

## Troubleshooting

### Bus Error Exception — Fa0/0 DHCP

During initial configuration, `ip address dhcp` was applied to Fa0/0 instead of Fa0/1. The router threw a Bus Error exception and crashed immediately.

```
*** System received a Bus Error exception ***
signal= 0xa, code= 0x200, context= 0x80e07d88
PC = 0x80024110, Vector = 0x200, SP = 0x8111aee8
```

![Cisco 2621 Bus Error exception and ROMMON boot sequence](../images/cisco-2600-boot-sequence-troubleshooting.jpg)

The router rebooted into ROMMON. Root cause: Fa0/0 had no physical cable — the Bus Error was triggered by the interface attempting DHCP negotiation with no physical link. Config was rebuilt on Fa0/1 where the cable was actually connected.

---

### Interface Fa0/0 vs Fa0/1 — IP Assignment

After ROMMON recovery, ping to `192.168.50.10` failed with `No route to host`.

```
NetOps-R1#show ip interface brief

Interface         IP-Address      OK? Method Status    Protocol
FastEthernet0/0   192.168.50.10   YES NVRAM  up        down
FastEthernet0/1   unassigned      YES NVRAM  admin down down
```

`up/down` on Fa0/0 = no physical link. Cable was on Fa0/1.

**Resolution:**

```
interface fastethernet0/0
 no ip address
 shutdown
exit
interface fastethernet0/1
 ip address 192.168.50.10 255.255.255.0
 no shutdown
```

Result:
```
FastEthernet0/1   192.168.50.10   YES manual up   up
```

---

## ROMMON Password Recovery

The original startup-config contained an unknown `enable secret` hash. Standard privileged exec access was blocked.

### Symptom

```
Router>enable
Password:
Password:
Password:
% Bad secrets
```

### Root Cause

`enable secret` (MD5 hash) present in startup-config from previous owner. Hash unknown — cannot be reversed. `enable secret` overrides `enable password` regardless of what is set in running-config.

### Recovery Procedure

**Step 1 — Send break signal during boot**

In `screen` on macOS, within 60 seconds of power-on:
```
Ctrl+A then \
```

**Step 2 — Set config register to bypass NVRAM**

```
rommon 1 > confreg 0x2142
rommon 2 > reset
```

`0x2142` instructs the router to ignore startup-config on boot. Router comes up with blank running-config — no passwords enforced.

**Step 3 — Enter privileged exec**

```
Router>enable
Router#
```

No password required — running-config is empty.

**Step 4 — Erase startup-config**

> **Critical:** Do not run `copy startup-config running-config`. This reloads the unknown `enable secret` hash back into RAM, blocking access again.

```
Router#erase startup-config
```

**Step 5 — Build config from scratch**

```
Router#conf t
Router(config)#hostname NetOps-R1
NetOps-R1(config)#enable password cisco123
NetOps-R1(config)#line vty 0 4
NetOps-R1(config-line)#password cisco
NetOps-R1(config-line)#login
NetOps-R1(config-line)#transport input telnet
NetOps-R1(config-line)#exit
NetOps-R1(config)#interface fastethernet0/1
NetOps-R1(config-if)#ip address 192.168.50.10 255.255.255.0
NetOps-R1(config-if)#no shutdown
NetOps-R1(config-if)#exit
NetOps-R1(config)#config-register 0x2102
NetOps-R1(config)#exit
NetOps-R1#write memory
NetOps-R1#reload
```

**Step 6 — Verify after reload**

```
show version
# Configuration register is 0x2102 — confirms normal boot restored

show ip interface brief
# FastEthernet0/1 — 192.168.50.10 — up — up
```

---

## IOS Upgrade — Decision Log

### Original Plan

Upgrade 2621 from `c2600-d-mz` (base, no crypto) to `c2600-adventerprisek9-mz` (crypto-capable) via TFTP to enable SSH.

### Flash Analysis

```
NetOps-R1#show flash
[5377816 bytes used, 11399400 available, 16777216 total]
```

| Item | Size |
|---|---|
| Current image (`c2600-d-mz`) | ~5.4MB |
| Available flash | ~11.4MB |
| Required k9 image | ~13–16MB |

Available flash insufficient to hold both images simultaneously. Upgrade requires delete-first procedure — if TFTP transfer fails mid-stream, router has no bootable image and requires ROMMON recovery with TFTP boot. Upgrade deferred. Cisco 1700 evaluated as alternative.

---

## Cisco 1700 — SSH Lab Device

### Discovery

```
Router>show version
Cisco IOS Software, C1700 Software (C1700-ADVSECURITYK9-M), Version 12.4(25d)
System image file is "flash:c1700-advsecurityk9-mz.124-25d.bin"

Router>show flash
[13726016 bytes used, 2789052 available, 16515068 total]
```

### Device Info

| Setting | Value |
|---|---|
| IOS Image | `c1700-advsecurityk9-mz.124-25d.bin` |
| IOS Version | 12.4(25d) |
| Crypto Support | Full — advsecurityk9 |
| Flash | 16MB — 2.7MB available |

> `advsecurityk9` feature set includes SSH, RSA key generation, and full crypto support.

### SSH Configuration — In Progress (Lab 04)

```
conf t
ip domain-name ccnahome.lab
username admin privilege 15 secret <password>
crypto key generate rsa
line vty 0 4
 transport input ssh
 login local
exit
write memory
```

---

## Console Access — Technical Notes

| Item | Detail |
|---|---|
| USB serial adapter | Insignia USB-A — `/dev/tty.usbserial-A9Q7R5CX` |
| Terminal command | `screen /dev/tty.usbserial-A9Q7R5CX 9600` |
| Break signal (macOS screen) | `Ctrl+A` then `\` |
| Ghost session check | `screen -list` |
| Kill ghost session | `screen -X -S [session name] quit` |
| 1700 console port | Labeled CON — not Ethernet or AUX |

> If `screen` terminates immediately, run `screen -list` and kill any detached sessions holding the port before opening a new one.

---

## Next Steps

- Configure SSH on Cisco 1700 — domain name, RSA keys, `transport input ssh`
- Connect 1700 to Linksys LAN — static IP in 192.168.50.x range
- Capture SSH session in Wireshark — compare vs Telnet cleartext
- Bring Catalyst 3500XL online — console access, IOS check, VLAN config
- Acquire and configure Raspberry Pi — TFTP server and lab endpoint
- Build Golden Config Auditor — Netmiko-based automation project

---

*Lavoisier Cornerstone — [lavoisier.dev](https://lavoisier.dev) | [github.com/cornerstonian](https://github.com/cornerstonian)*

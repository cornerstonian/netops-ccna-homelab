# NetOps CCNA Safety Buffer Homelab (Physical)

**Goal:** Isolated physical CCNA lab environment with internet access — primary home network fully protected.

![Lab Setup](images/homelab-setup.jpg)

---

## Overview

- Cisco 2621 (NetOps-R1) configured as lab gateway behind a Linksys E2500 buffer router
- Network isolation prevents lab misconfigurations from reaching the primary home network
- Static IP management addressing on Fa0/1 — converted from initial DHCP assignment
- Telnet remote management active on 2621
- Cisco 1700 confirmed crypto-capable (`advsecurityk9` IOS 12.4(25d)) — SSH configuration in progress

---

## Equipment & Technologies

| Device | Role | Status |
|---|---|---|
| Cisco 2621 — NetOps-R1 | Lab gateway — IOS 12.1(3)T | ✅ Active |
| Cisco 1700 | SSH lab device — IOS 12.4(25d) advsecurityk9 | ✅ Active |
| Catalyst 3500XL Switch | Core lab LAN switch | 📋 Planned |
| Linksys E2500 | Safety buffer router — NAT + DHCP | ✅ Active |
| MacBook Pro 2015 | Console management / capture station | ✅ Active |
| Raspberry Pi | TFTP server / Linux endpoint / automation | 📋 Planned |

---

## High-Level Topology

![Topology](images/homelab-topology.png)

```
Internet
   ↓
Xfinity Gateway — 192.168.1.x
   ↓
Linksys E2500 — 192.168.50.1 (NAT + DHCP)
   ↓
   ├── Cisco 2621 — NetOps-R1 — 192.168.50.10 (Fa0/1)
   └── Cisco 1700 — 192.168.50.x (in progress)

MacBook — 192.168.50.147 (en4) — console + capture
```

---

## Key Achievements

| Achievement | Detail |
|---|---|
| Lab isolation | Linksys E2500 buffer prevents lab traffic from reaching `192.168.1.x` |
| Static management IP | 2621 Fa0/1 converted from DHCP to `192.168.50.10` |
| Internet reachability | Verified via ping to `8.8.8.8` through full lab chain |
| Telnet remote access | VTY 0-4 configured on 2621 |
| ROMMON password recovery | Unknown `enable secret` recovered — startup-config erased and rebuilt |
| SSH device identified | Cisco 1700 running `advsecurityk9` IOS 12.4(25d) — SSH ready |

---

## Next Steps

- Configure SSH on Cisco 1700 — RSA keys, `transport input ssh`
- Connect 1700 to Linksys LAN — static IP in 192.168.50.x range
- Bring Catalyst 3500XL online — VLAN configuration
- Acquire and configure Raspberry Pi — TFTP server and lab endpoint
- Build Golden Config Auditor — Netmiko automation project

---

## Repository Structure

```
configs/              ← router running config backups
images/               ← lab photos
project-writeups/     ← per-project technical documentation
topology-diagrams/    ← network topology diagrams
troubleshooting/      ← standalone issue documentation
```

## Documentation

| Document | Description |
|---|---|
| [secure-ccna-lab-integration.md](project-writeups/secure-ccna-lab-integration.md) | Full technical writeup — lab build, configs, ROMMON recovery, troubleshooting |

---

*Lavoisier Cornerstone — [lavoisier.dev](https://lavoisier.dev) | [github.com/cornerstonian](https://github.com/cornerstonian)*

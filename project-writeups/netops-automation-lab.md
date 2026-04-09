# NetOps Automation Lab — Technical Writeup

**Repo:** [cornerstonian/netops-ccna-homelab](https://github.com/cornerstonian/netops-ccna-homelab)
**Status:** 🔄 In Progress — Netmiko installed, scripting not yet begun
**Last Updated:** April 2026

---

## Lab Goals

| Goal | Status |
|---|---|
| Install Netmiko on MacBook | ✅ Complete |
| Configure Telnet on NetOps-SW1 for automation access | 📋 Planned |
| Connect to NetOps-1700 via SSH using Netmiko | 📋 Planned |
| Connect to NetOps-R1 via Telnet using Netmiko | 📋 Planned |
| Connect to NetOps-SW1 via Telnet using Netmiko | 📋 Planned |
| Automate `show ip interface brief` across all devices | 📋 Planned |
| Build config backup script — timestamped pulls to `configs/` | 📋 Planned |
| Build interface compliance checker — verify VLAN topology is intact | 📋 Planned |
| Document all scripts with inline comments | 📋 Planned |
| Push automation scripts to `automation/` directory | 📋 Planned |

---

## Hardware & Access

| Device | Protocol | Netmiko device_type | IP | Status |
|---|---|---|---|---|
| NetOps-1700 | SSH | `cisco_ios` | 192.168.20.20 | ✅ Ready |
| NetOps-R1 | Telnet | `cisco_ios_telnet` | 192.168.10.1 | ✅ Ready |
| NetOps-SW1 | Telnet | `cisco_ios_telnet` | 192.168.50.30 | 📋 Telnet config needed |

---

## Environment

| Item | Detail |
|---|---|
| Language | Python 3.11 |
| Library | Netmiko 4.6.0 |
| Supporting libraries | Paramiko 4.0.0, ntc-templates 9.1.0, textfsm 2.1.0 |
| Scripts directory | `automation/` |
| Config output directory | `configs/` |
| MacBook interface | en4 — 192.168.10.50 (VLAN 10) |

---

## Planned Scripts

### Script 1 — Single Device Connection Test (`connect_test.py`)

Basic Netmiko connection to NetOps-1700 via SSH. Runs `show ip interface brief` and prints output. Verifies Netmiko can negotiate legacy IOS 12.4 SSH algorithms via Paramiko.

> **Known friction point:** `~/.ssh/config` handles the macOS OpenSSH client but Netmiko uses Paramiko — a separate SSH library with its own algorithm negotiation. May require `disabled_algorithms` parameter passed explicitly in the device dict.

### Script 2 — Multi-Device Show Commands (`collect_show.py`)

Loops across all three devices (SSH for 1700, Telnet for R1 and SW1), runs `show ip interface brief` on each, and prints labeled output. First script to exercise all three devices simultaneously.

### Script 3 — Config Backup (`collect_configs.py`)

Pulls `show running-config` from all devices and saves to `configs/` with timestamped filenames. Intended to run before every lab session as a restore-point snapshot.

```
configs/
   NetOps-R1-20260408-1430.txt
   NetOps-1700-20260408-1430.txt
   NetOps-SW1-20260408-1430.txt
```

### Script 4 — Interface Compliance Checker (`check_interfaces.py`)

SSHes/Telnets into each device, runs `show ip interface brief`, parses output, and verifies expected interfaces are up/up. Alerts if any interface is down or missing. Validates the Lab 05 VLAN topology is intact before starting a lab session.

Expected state it checks:

| Device | Interface | Expected IP | Expected State |
|---|---|---|---|
| NetOps-R1 | Fa0/0.10 | 192.168.10.1 | up/up |
| NetOps-R1 | Fa0/0.20 | 192.168.20.1 | up/up |
| NetOps-1700 | Fa0 | 192.168.20.20 | up/up |
| NetOps-SW1 | VLAN1 | 192.168.50.30 | up/up |

---

## CCNA Exam Alignment

This lab maps directly to CCNA 200-301 Domain 6.0 — Automation and Programmability.

| Exam Topic | Lab Coverage |
|---|---|
| 6.1 — Explain how automation impacts network management | Config backup script demonstrates automated vs manual config pulls |
| 6.2 — Compare traditional vs controller-based networking | Netmiko represents script-based automation — contrast with intent-based (Ansible, NSO) |
| 6.3 — Describe controller-based architectures | Covered conceptually in writeup |
| 6.5 — Interpret basic Python script for network automation | All four scripts are directly interpretable examples |
| 6.6 — Describe APIs for network management | Netmiko uses SSH/Telnet APIs — foundation for REST API understanding |

---

## Notes

- Netmiko installed April 2026 — `pip3 install netmiko` — version 4.6.0
- Scripting begins next session
- 3500XL Telnet config required before SW1 can be automated — two commands on the switch console

---

*Lavoisier Cornerstone — [lavoisier.dev](https://lavoisier.dev) | [github.com/cornerstonian](https://github.com/cornerstonian)*

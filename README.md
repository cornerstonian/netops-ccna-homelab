# NetOps CCNA Safety Buffer Homelab (Physical)

Goal: Build an isolated CCNA lab environment that allows internet access while protecting the primary home network.

![homelab-documentation-photo-](images/ccna-lab-physical-setup-documentation.jpg)


## High-Level Overview

- Cisco 2600 configured as a lab gateway behind a Linksys E2500 buffer router
- Network isolation implemented to protect the primary home network
- DHCP connectivity established and internet reachability verified
- Router prepared for SSH remote management

## Equipment & Technologies

- Cisco 2600 Router (Lab Gateway)
- Cisco 1700 Router
- Catalyst 3500 XL Switch
- Linksys E2500 Router (Safety Buffer)
- MacBook Terminal (Console Access)
- Raspberry Pi (planned endpoint)

## High-Level Topology

![Topology](topology-diagrams/netops-lab-topology.png)

### Key Achievements

• Implemented a safety buffer network to isolate CCNA lab traffic  
• Configured Cisco 2600 router connectivity behind a Linksys E2500 buffer router  
• Converted WAN interface from DHCP to static IP for stable management addressing  
• Configured default routing to the upstream gateway  
• Verified Layer 3 connectivity to the buffer router  
• Implemented Telnet-based remote management via VTY lines  
• Identified IOS feature limitations preventing SSH configuration

### Next Steps

• Upgrade Cisco 2600 IOS image via TFTP to enable crypto/SSH features
• Connect the Catalyst 3500XL switch
• Configure the internal lab LAN
• Integrate Raspberry Pi endpoint for TFTP and automation testing

## Full Lab Documentation

Detailed step-by-step documentation is available here:

[project-writeups/secure-ccna-lab-integration.md
](https://github.com/cornerstonian/netops-ccna-homelab/blob/main/project-writeups/secure-ccna-lab-integration.md#lab-goals)




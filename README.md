# Home Data Center Lab

A hands-on home lab simulating enterprise data center infrastructure — built on real hardware to practice network segmentation, Linux server administration, and incident troubleshooting relevant to Data Center Technician / IT Support roles.

## Overview

This project recreates core data center operations at a small scale: physical networking, VLAN segmentation, inter-VLAN routing, and a Linux server hosting multiple production-style services. It also includes a documented incident log from deliberately simulated failure scenarios — the kind of day-to-day troubleshooting a data center technician actually performs.

## Topology

```
                    Internet
                       |
                Verizon Router
                       |
              TP-Link TL-SG608E
              (Managed Switch)
                 /          \
          Access Port    Trunk Port
          (VLAN 10)     (VLAN 10 + 20, tagged)
              |               |
          Laptop         Dell Server
          (Windows)      (Ubuntu Server)
       192.168.10.10   vlan10: 192.168.10.1
                        vlan20: 192.168.20.1
                        (router-on-a-stick)
```

## Equipment

| Component | Role |
|---|---|
| Dell Desktop | Ubuntu Server — router + application server |
| ASUS TUF Laptop | Windows client / admin workstation |
| TP-Link TL-SG608E | 8-port managed switch (802.1Q VLAN capable) |
| Verizon Router | Internet gateway |
| Cat6e Ethernet cabling | Physical connectivity |
| USB Flash Drive | Bootable Ubuntu Server installer (via Rufus) |

## Skills Demonstrated

- **Layer 1 (Physical):** Cabling, link verification, cable fault diagnostics
- **Layer 2 (Switching):** 802.1Q VLAN configuration, access/trunk ports, port-level administration
- **Layer 3 (Routing):** Inter-VLAN routing via Linux router-on-a-stick, static routing, IP forwarding, subnetting
- **Linux Server Administration:** Ubuntu Server installation, `systemctl` service management, Netplan network configuration
- **Infrastructure Services:** SSH, Apache (web server), Samba (SMB file sharing), Docker (containerization)
- **Troubleshooting Methodology:** Hypothesis-driven diagnosis, log analysis, root cause identification, documented resolution

## Build Log

### Phase 1 — Physical Layer
Cabled router → switch → server/laptop. Verified link status via port LEDs and the switch's built-in cable diagnostics tool.

### Phase 2 — VLAN Segmentation
Configured two 802.1Q VLANs on the managed switch:
- **VLAN 10 (OFFICE)** — laptop, access port
- **VLAN 20 (SERVER)** — Dell server, access port (later reconfigured to trunk)

Verified isolation — confirmed no cross-VLAN connectivity prior to routing configuration.

### Phase 3 — Inter-VLAN Routing (Router-on-a-Stick)
Since the server has a single physical NIC, implemented router-on-a-stick:
- Reconfigured the server's switch port as an 802.1Q trunk (tagged VLAN 10 + 20)
- Created VLAN sub-interfaces on Ubuntu via Netplan:
  ```yaml
  network:
    vlans:
      vlan10:
        id: 10
        link: eno1
        addresses: [192.168.10.1/24]
      vlan20:
        id: 20
        link: eno1
        addresses: [192.168.20.1/24]
  ```
- Enabled IP forwarding: `sysctl -w net.ipv4.ip_forward=1`
- Verified full inter-VLAN connectivity from laptop through server to both network segments

### Phase 4 — Ubuntu Server Build
Installed Ubuntu Server 26.04 LTS. Resolved a UEFI/Legacy boot mismatch in BIOS post-install.

Deployed and verified:
- **SSH** — remote administration (`systemctl status ssh`)
- **Apache2** — web service, verified reachable across VLAN routing (`http://192.168.20.1`)
- **Samba** — SMB file share (`/srv/samba/shared`), authenticated access, verified with live file transfer test
- **Docker** — containerization engine, verified with `docker run hello-world`

## Incident Log

Four documented failure scenarios, each broken intentionally, diagnosed, resolved, and logged in [`tickets.md`](./tickets.md).:

| Ticket | Scenario | Root Cause | Resolution |
|---|---|---|---|
| #001 | Switch port disabled | Port administratively disabled via switch config | Re-enabled port in switch management interface |
| #002 | IP misconfiguration | Static IP and gateway placed in different subnets | Corrected IP to match gateway's subnet |
| #003 | SSH service down | `ssh.service` and `ssh.socket` both stopped | Restarted both units via `systemctl` |
| #004 | Disk space exhaustion | Oversized test file consumed 95% of disk | Identified and removed file via `du`/`rm` |


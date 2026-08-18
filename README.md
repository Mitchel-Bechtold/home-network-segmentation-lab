# Enterprise Home Network with Validated Segmentation

A multi-VLAN segmented home network with stateful firewall policy, DNS-layer threat filtering, and validated segmentation through internal penetration testing. Built to demonstrate practical network engineering and defense-in-depth security architecture.

![Network Topology](diagrams/network-topology.png)

## Project Overview

This project implements enterprise network architecture in a home environment, isolating four trust-tier segments using 802.1Q VLANs with policy enforced at a pfSense firewall. Network segmentation was validated through hostile-position penetration testing using Kali Linux.

## Architecture

### VLAN Design

| VLAN | Name | Subnet | Trust Level | Purpose |
|------|------|--------|-------------|---------|
| 10 | Trusted | 192.168.10.0/24 | High | Personal devices |
| 20 | IoT | 192.168.20.0/24 | Low | Smart home, untrusted devices |
| 30 | Guest | 192.168.30.0/24 | None | Visitor internet only |
| 40 | Lab | 192.168.40.0/24 | Hostile | Security research |

### Security Controls

- **802.1Q VLAN segmentation** — Layer 2 broadcast domain isolation
- **Subnet isolation** — Layer 3 routing forces inter-VLAN traffic through firewall
- **Stateful firewall policy** — Per-VLAN access rules enforcing trust-tier model
- **DNS-layer threat filtering** — pfBlockerNG with multi-source threat feeds
- **Management plane separation** — Dedicated VLAN for network administration

## Hardware

- **Firewall:** Dell OptiPlex 7040 SFF + Intel I350-T2 dual NIC
- **Switch:** TP-Link TL-SG108E (8-port managed, 802.1Q capable)
- **Access Point:** TP-Link EAP225 (PoE, multi-SSID with VLAN tagging)
- **Software:** pfSense Community Edition 2.7.x

## Validation

Network segmentation was validated through internal penetration testing using Kali Linux deployed on the Lab VLAN, simulating a hostile insider or compromised device. Standard reconnaissance tools (nmap, arp-scan) confirmed:

- Zero cross-VLAN host discovery from Lab VLAN
- pfSense management interfaces unreachable from hostile position
- Layer 2 broadcast domain properly isolated per VLAN
- DNS-layer threat filtering applies across all segments

See [docs/08-segmentation-validation.md](docs/08-segmentation-validation.md) for complete test results.

## Documentation

- [Design and Architecture](docs/01-design-architecture.md)
- [Hardware Bill of Materials](docs/02-hardware-bom.md)
- [pfSense Installation](docs/03-pfsense-installation.md)
- [VLAN and Switch Configuration](docs/04-vlan-switch-config.md)
- [Wireless VLAN Tagging](docs/05-wireless-vlan-tagging.md)
- [Firewall Policy](docs/06-firewall-policy.md)
- [DNS Filtering](docs/07-dns-filtering.md)
- [Segmentation Validation](docs/08-segmentation-validation.md)
- [Lessons Learned](docs/09-lessons-learned.md)

## Skills Demonstrated

- Network design and VLAN architecture
- pfSense administration and firewall policy design
- 802.1Q tagging — trunk and access port configuration
- Subnetting and IP planning (RFC 1918)
- Enterprise WiFi with per-SSID VLAN assignment
- DNS-layer security filtering
- Penetration testing methodology
- Linux/FreeBSD CLI fundamentals
- Technical documentation and architecture diagramming

## Status

  Project complete — 4 VLANs operational, firewall policy enforced, DNS filtering active, segmentation validated.

  Future enhancements:
- Suricata IDS/IPS deployment
- Wazuh SIEM integration for centralized log analysis (Project 2)
- Active Directory attack lab integration (Project 3)

## Author

Mitchel Bechtold | https://www.linkedin.com/in/mitchel-bechtold-a321a2250/ | Contact information: mcmitchel@outlook.com

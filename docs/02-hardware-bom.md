# Hardware Bill of Materials

Components selected for the home network lab, with rationale for each choice.

## Total Investment

Approximately $200 in hardware (used market pricing).

## Component List

### Firewall Appliance: Dell OptiPlex 7040 SFF

| Spec | Value |
|------|-------|
| CPU | Intel Core i5-6500 (6th gen Skylake) |
| RAM | 8 GB DDR4 |
| Storage | SSD 1TB in size |
| Form Factor | Small Form Factor (SFF) |
| Approximate Cost | $40 (used) |

**Rationale:** The OptiPlex 7040 is a popular pfSense platform for several reasons:

- **Adequate CPU for the workload:** pfSense's official minimum is a 1 GHz CPU and 1 GB RAM. The 6th-gen i5 vastly exceeds this and provides headroom for Suricata IDS, pfBlockerNG, and VPN throughput.
- **Low power consumption:** ~15-25W idle makes it suitable for 24/7 operation.
- **PCIe slots for NIC expansion:** The SFF form factor includes a half-height PCIe slot, allowing addition of a proper NIC for the WAN interface.
- **Reliable used market availability:** Corporate retirement floods the used market with these machines at $80-130.

### Network Interface Card: Intel I350-T2

| Spec | Value |
|------|-------|
| Ports | 2 × Gigabit Ethernet |
| Chipset | Intel I350 |
| Bus | PCIe x4 |
| Approximate Cost | $15 (used) |

**Rationale:** The OptiPlex 7040 has only one onboard NIC, but pfSense requires at least two interfaces (WAN and LAN). The Intel I350 series is the de facto standard for pfSense for several reasons:

- **First-class FreeBSD driver support:** Intel NICs work flawlessly with pfSense's underlying FreeBSD OS, unlike Realtek-based cards which have known driver issues.
- **Server-grade reliability:** Designed for enterprise use, more reliable than consumer-grade alternatives.
- **Hardware offload capabilities:** Reduces CPU load for high-throughput scenarios.

### Managed Switch: TP-Link TL-SG108E

| Spec | Value |
|------|-------|
| Ports | 8 × Gigabit Ethernet |
| Management | Web-managed (Easy Smart) |
| VLAN Support | 802.1Q with PVID |
| Approximate Cost | $27 |

**Rationale:** Multi-VLAN segmentation requires a switch capable of 802.1Q tagging. The TL-SG108E is one of the lowest-cost options that supports the full 802.1Q feature set including:

- Tagged trunk ports for multi-VLAN backbone connections
- Untagged access ports for end-device connections
- Per-port PVID configuration
- 16 simultaneous VLANs supported

The TP-Link Easy Smart UI is dated but functional. Configuration requires explicit "Save Config" action — changes are lost on power cycle if not saved to flash.

### Wireless Access Point: TP-Link EAP225

| Spec | Value |
|------|-------|
| WiFi Standard | 802.11ac Wave 2 |
| Bands | Dual-band (2.4 + 5 GHz) |
| Power | PoE (802.3af) |
| Approximate Cost | $60 |

**Rationale:** Multi-VLAN wireless requires per-SSID VLAN tagging — a feature absent from most consumer routers but standard on enterprise APs. The EAP225 supports:

- Up to 8 SSIDs per radio
- Per-SSID VLAN ID assignment
- Standalone web UI (no controller required for single AP deployment)
- PoE power delivery (single cable for power and data)

### Cables and Accessories

- 2× Cat6 patch cable, 10 ft (switch trunk to pfSense)
- 1× Cat6 patch cable, 50 ft (switch to AP, in-wall run)
- Velcro cable ties for organization

## Hardware NOT Used (And Why)

### Raspberry Pi for pfSense

**Considered but rejected.** The Raspberry Pi has:
- Only one onboard NIC (would require USB-to-Ethernet adapter for second interface, unreliable for 24/7 router use)
- ARM architecture (pfSense is x86-64, not officially supported on ARM)
- Lower throughput ceiling than even an old x86 mini PC

A used Dell OptiPlex provides better performance at similar cost.

### Consumer All-in-One Router

**Considered but rejected.** Consumer routers (Asus, Netgear, Linksys) typically lack:
- Per-port VLAN configuration
- Per-SSID VLAN tagging
- Detailed firewall rule logging
- Custom DNS filtering with feed management

Using dedicated firewall, switch, and AP roles provides the flexibility required for a true segmented network.

### Cloud-Managed Wireless (UniFi, Aruba Instant On)

**Considered but accepted alternative.** The TP-Link Omada platform offers similar functionality. EAP225 was selected based on cost; UniFi nanoHD or U6-Lite would also have worked.

## Summary

Total hardware cost (used market pricing): approximately $200.

This stack provides enterprise-grade network segmentation capability at a fraction of new commercial hardware cost, suitable for a home environment while remaining representative of small business deployments.
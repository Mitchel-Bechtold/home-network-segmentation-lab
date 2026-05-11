# VLAN and Switch Configuration

Configuration of 802.1Q VLAN tagging across pfSense and the TP-Link TL-SG108E switch.

## Concepts Quick Reference

- **VLAN (802.1Q tagging):** A 4-byte tag inserted into Ethernet frames identifying which logical network the frame belongs to
- **Trunk port:** Carries multiple VLANs with tags preserved; used between VLAN-aware devices
- **Access port:** Carries one VLAN with tags stripped; used for end devices
- **PVID (Port VLAN ID):** Tells the switch which VLAN to assign untagged frames arriving on a port

## Network Plan

### VLANs Defined

| VLAN ID | Name | Subnet | Gateway |
|---------|------|--------|---------|
| 10 | TRUSTED | 192.168.10.0/24 | 192.168.10.1 |
| 20 | UNTRUSTED (IoT) | 192.168.20.0/24 | 192.168.20.1 |
| 30 | GUEST | 192.168.30.0/24 | 192.168.30.1 |
| 40 | LAB | 192.168.40.0/24 | 192.168.40.1 |

VLAN 1 is reserved for management traffic (pfSense LAN interface, switch management, AP management).

### Switch Port Plan

| Port | Role | VLAN(s) | Tag State | PVID |
|------|------|---------|-----------|------|
| 1 | Trunk → pfSense | 1, 10, 20, 30, 40 | 1 untagged, others tagged | 1 |
| 2 | Trunk → AP | 1, 10, 20, 30, 40 | 1 untagged, others tagged | 1 |
| 3 | TRUSTED access | 10 | Untagged | 10 |
| 4 | TRUSTED access | 10 | Untagged | 10 |
| 5 | LAB access | 40 | Untagged | 40 |
| 6 | LAB access | 40 | Untagged | 40 |
| 7 | UNTRUSTED access | 20 | Untagged | 20 |
| 8 | Management access | 1 | Untagged | 1 |

## pfSense Configuration

### Create VLAN Interfaces

In pfSense web UI: **Interfaces → Assignments → VLANs tab**

Created four VLAN interfaces, all on parent interface `em0` (LAN):

| Parent | VLAN Tag | Description |
|--------|----------|-------------|
| em0 | 10 | TRUSTED |
| em0 | 20 | UNTRUSTED |
| em0 | 30 | GUEST |
| em0 | 40 | LAB |

### Assign VLAN Interfaces

In **Interfaces → Assignments**, added each VLAN as a network interface (creating OPT1 through OPT4), then renamed each via the interface configuration page.

### Configure Each Interface

For each VLAN interface:

- Enabled
- IPv4 Configuration Type: Static IPv4
- IPv4 Address: 192.168.[VLAN].1 / 24
- IPv6: None

### Enable DHCP Per VLAN

In **Services → DHCP Server**, enabled DHCP on each VLAN interface:

| VLAN | DHCP Range |
|------|-----------|
| TRUSTED | 192.168.10.100 - 192.168.10.200 |
| UNTRUSTED | 192.168.20.100 - 192.168.20.200 |
| GUEST | 192.168.30.100 - 192.168.30.200 |
| LAB | 192.168.40.100 - 192.168.40.200 |

DNS Servers field left blank — devices use the gateway IP of their VLAN as DNS server (pfSense's resolver).

## TP-Link Switch Configuration

### Initial Access

The TL-SG108E ships with default IP 192.168.0.1 but obtained an address via DHCP from pfSense at 192.168.1.X. Located its address via Status → DHCP Leases on pfSense.

Set switch to static IP 192.168.1.2 for stability.

### Enable 802.1Q VLAN Mode

**VLAN → 802.1Q VLAN → Enable** (the master switch for VLAN functionality).

### Create VLANs

For each VLAN, configured tagged and untagged port memberships:

**VLAN 10 (TRUSTED):**
- Untagged Ports: 3, 4
- Tagged Ports: 1, 2

**VLAN 20 (UNTRUSTED):**
- Untagged Ports: 7
- Tagged Ports: 1, 2

**VLAN 30 (GUEST):**
- Untagged Ports: (none — wireless only)
- Tagged Ports: 1, 2

**VLAN 40 (LAB):**
- Untagged Ports: 5, 6
- Tagged Ports: 1, 2

### Configure PVID

In **VLAN → 802.1Q PVID Setting**, set the default VLAN for untagged frames per port:

| Port | PVID |
|------|------|
| 1 | 1 (trunk) |
| 2 | 1 (trunk) |
| 3 | 10 |
| 4 | 10 |
| 5 | 40 |
| 6 | 40 |
| 7 | 20 |
| 8 | 1 |

PVID is critical — without correct PVID, untagged traffic from end devices ends up in the wrong VLAN regardless of the access port configuration. This is the most commonly missed step.

### Save Configuration

TP-Link Easy Smart switches require explicit "Save Config" action — changes are stored only in RAM until saved to flash. **System → Save Config** writes the running configuration to nonvolatile storage.

Without this step, all configuration is lost on reboot or power loss.

## Verification

Tested wired VLAN segmentation by plugging a device into each access port and verifying:

1. Device received DHCP address in correct subnet for that VLAN
2. Device could reach internet
3. Device could reach pfSense at the gateway IP for its VLAN

Sample test results:

| Port plugged into | DHCP IP received | VLAN |
|-------------------|------------------|------|
| Port 3 | 192.168.10.137 | TRUSTED ✓ |
| Port 5 | 192.168.40.142 | LAB ✓ |
| Port 7 | 192.168.20.118 | UNTRUSTED ✓ |
| Port 8 | 192.168.1.50 | Management ✓ |

All access ports correctly assigning devices to intended VLANs.

## Troubleshooting Notes

### Devices Not Receiving Correct VLAN IP

The most common issue. Caused by incorrect PVID. Verified PVID matches the untagged VLAN of the port.

### Lost Configuration on Reboot

TP-Link Easy Smart switches don't auto-save. After every batch of changes, must explicitly Save Config to flash.

### Could Not Access Switch UI from Other VLANs

Many embedded devices (including this switch) only respond to web management traffic from devices on the same subnet. Mac on TRUSTED could not reach switch at 192.168.1.2; required moving Mac to a management VLAN access port.

This led to the addition of port 8 as a dedicated management access port — typed into the design specifically to enable network admin access without re-cabling.

## References

Screenshots of switch and pfSense VLAN configuration in `../screenshots/switch/` and `../screenshots/vlan-config/`.
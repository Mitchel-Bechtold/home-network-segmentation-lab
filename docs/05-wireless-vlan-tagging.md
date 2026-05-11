# Wireless VLAN Tagging

Configuration of the TP-Link EAP225 access point with multiple SSIDs, each tagged to a different VLAN.

## Concept

A single physical AP broadcasts multiple SSIDs. Each SSID is associated with a VLAN ID at the AP level. When a wireless device connects to a specific SSID, the AP tags traffic from that device with the configured VLAN ID and forwards it via 802.1Q trunking to pfSense.

This means wireless devices are automatically segmented based on which SSID they connect to — no per-device configuration required.

## SSID Plan

| SSID | VLAN ID | Purpose | Bands |
|------|---------|---------|-------|
| HomeNet-Trusted | 10 | Personal devices | 2.4 + 5 GHz |
| HomeNet-IoT | 20 | Smart home, IoT | 2.4 GHz only |
| HomeNet-Guest | 30 | Visitors | 2.4 + 5 GHz |

No SSID for Lab — Lab is wired-only for security (prevents accidental wireless attachment).

### Why Same SSID on Both Bands

For Trusted and Guest, the same SSID is broadcast on both 2.4 GHz and 5 GHz with identical password and security settings. Modern devices automatically pick the optimal band based on signal strength.

If different SSIDs were used per band (e.g., "HomeNet-Trusted-5G"), users would have to manually choose, and devices wouldn't roam between bands.

### Why IoT Is 2.4 GHz Only

Most IoT devices (smart bulbs, plugs, older smart home gear) are 2.4 GHz only. Broadcasting IoT on 5 GHz wastes airtime and confuses devices that scan for compatible networks.

## AP Configuration

### Power and Connectivity

The EAP225 is PoE-powered via the TP-Link switch port 2 (the trunk port). Single Cat6 cable provides both power and data.

### Initial Access

After factory reset (10-second hold on reset pinhole), AP requested DHCP from pfSense and obtained 192.168.1.3 (via DHCP reservation configured in pfSense for stable management).

Accessed standalone web UI at `http://192.168.1.3`. Default credentials `admin`/`admin` immediately changed to a strong password.

### Standalone vs. Controller

The EAP225 supports two management modes:

- **Standalone:** Direct web UI on the AP itself
- **Omada Controller:** Centralized management via TP-Link's controller software

For a single AP, standalone is simpler and exposes all needed functionality including per-SSID VLAN tagging. Controller mode is more useful for multi-AP deployments.

This deployment uses standalone mode.

### SSID Configuration

In **Wireless → Wireless Settings**, created three SSIDs.

For each SSID:
- SSID name (as shown in plan above)
- Security: WPA2-Personal (PSK)
- Strong unique password (saved in password manager)
- VLAN: Enabled
- VLAN ID: matching the trust tier

Additional settings:

- **HomeNet-Guest:** Client Isolation enabled (guests cannot see each other on the wireless network)
- **HomeNet-IoT:** 2.4 GHz only

## Verification

Connected a phone to each SSID and verified DHCP-assigned IP:

| SSID Connected To | IP Received | Expected | Result |
|-------------------|-------------|----------|--------|
| HomeNet-Trusted | 192.168.10.X | 192.168.10.X | ✓ |
| HomeNet-IoT | 192.168.20.X | 192.168.20.X | ✓ |
| HomeNet-Guest | 192.168.30.X | 192.168.30.X | ✓ |

Wireless segmentation working as designed — devices automatically placed in the correct VLAN based on SSID selection.

## Troubleshooting Notes

### AP Not Discoverable by Omada Controller

Initial attempt to use the Omada Software Controller via Docker failed because the controller was on a different VLAN than the AP. Omada uses Layer 2 broadcast for AP discovery, which doesn't cross VLAN boundaries.

Resolved by abandoning controller approach and using the AP's standalone web UI directly. Standalone web UI provides equivalent functionality for a single AP.

### Standalone Web UI Disabled After Adoption

Once the AP was adopted by the mobile Omada app, the standalone web UI became inaccessible. Resolved via factory reset, which restored standalone web UI access.

Lesson: Once an AP is managed by a controller (mobile app or otherwise), it can only be managed by that controller. Removing controller management requires factory reset.

### Mobile App Missing VLAN Settings

The TP-Link Omada mobile app's "Standalone Mode" hides per-SSID VLAN configuration. This is a deliberate limitation by TP-Link — VLAN tagging requires either the standalone web UI or the full Omada controller.

Resolved by using the standalone web UI directly.

## References

Screenshots of AP configuration in `../screenshots/ap/` and wireless validation in `../screenshots/wireless-validation/`.
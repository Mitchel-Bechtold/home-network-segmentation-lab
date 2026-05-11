# DNS Filtering with pfBlockerNG

DNS-layer filtering of advertising, tracking, malware, and phishing domains using pfBlockerNG.

## Concept

DNS filtering blocks domain resolution before connections are attempted. When a device queries a known-bad domain, the DNS resolver returns a sinkhole IP instead of the real address. The device's connection attempt fails because it can't reach the sinkhole.

This blocks threats before they can establish connections, regardless of the device's own security posture. Effective even against IoT devices that can't run their own anti-malware.

## Why Layer DNS Filtering on Top of VLAN Segmentation

VLAN segmentation prevents lateral movement once a device is compromised. DNS filtering reduces the chance of compromise in the first place by blocking access to malicious domains.

Defense in depth: each layer addresses different threats, and they compound.

## pfBlockerNG Installation

Installed pfBlockerNG-devel via System → Package Manager → Available Packages.

The `-devel` branch is actively maintained; the older non-devel branch is deprecated.

## Configuration

### Setup Wizard

Ran the wizard at Firewall → pfBlockerNG → Wizard. Default values for most settings:

- Inbound Firewall Interface: WAN
- Outbound Firewall Interface: LAN
- VLAN configuration: defaults

### DNSBL Virtual IP

The wizard required a Virtual IP for DNSBL responses. Created via Firewall → Virtual IPs:

- Type: IP Alias
- Interface: Localhost
- Address: 10.10.10.1/32
- Description: pfBlockerNG DNSBL VIP

This IP is the destination returned for blocked DNS queries. Devices attempting to connect to it fail (nothing is listening), accomplishing the block.

### DNSBL Configuration

In Firewall → pfBlockerNG → DNSBL:

- Enable DNSBL: checked
- DNSBL Mode: Unbound python mode (more efficient than legacy mode)

### Block List Feeds

Configured DNSBL groups to subscribe to threat intelligence feeds:

| Feed | Source | Coverage |
|------|--------|----------|
| StevenBlack Unified | github.com/StevenBlack/hosts | Aggregated ads, trackers, malware |
| URLhaus | abuse.ch | Active malware C2 servers |
| Block List Project Phishing | blocklistproject.github.io | Phishing domains |

These feeds are community-maintained and updated regularly. They aggregate from primary threat intelligence sources (URLhaus, Spamhaus, Phishtank, EasyList, etc.) to produce comprehensive block lists.

This represents the open-source/homelab tier of threat intelligence — conceptually equivalent to enterprise feeds from vendors like Recorded Future or Mandiant, at no cost.

### Forced Update

After configuration, ran Firewall → pfBlockerNG → Update → Reload: All → Force checkbox → Run. This downloaded all feeds and built the active block list.

Initial sync took approximately 5-10 minutes.

## Validation

### nslookup of Known-Blocked Domain

- $ nslookup doubleclick.net
- Server:   192.168.X.1
- Name:     doubleclick.net
- Address:  0.0.0.0
- Returns 0.0.0.0 instead of a real IP — block working.

### Browser Test

Pages with ads load with blank rectangles where ads were. Page functionality unaffected.

### Reports Dashboard

Firewall → pfBlockerNG → Reports shows:
- Total blocked queries
- Top blocked domains
- Top querying clients
- Block rate over time

After several hours of typical use, thousands of blocked queries were logged — most ads and tracking, some legitimately malicious.

## Per-VLAN Application

DNS filtering applies to all VLANs that use pfSense as their DNS resolver, which is all four (Trusted, IoT/Untrusted, Guest, Lab). The firewall rule allowing DNS to pfSense (port 53) routes queries through pfBlockerNG's filtering before responding.

This means even IoT devices and guests benefit from threat filtering, which is significant — these devices typically lack other anti-malware protection.

## Update Schedule

pfBlockerNG automatically updates feeds on a schedule (default: daily). New malicious domains added by upstream feeds are typically blocked within 24 hours of being identified.

## Troubleshooting Notes

### Wizard Required Virtual IP

Initial wizard run failed because the DNSBL Virtual IP was not pre-configured. Required exiting wizard, creating the VIP via Firewall → Virtual IPs, then re-running wizard.

### False Positives

Some legitimate services use domains in the block lists (e.g., certain Microsoft telemetry endpoints). For each false positive:

- Identify via Reports tab — shows what's being blocked
- Add to whitelist: Firewall → pfBlockerNG → DNSBL → DNSBL Whitelist

Started with conservative feed selection (StevenBlack only) to minimize false positives, expanded to other feeds gradually.

## References

Screenshots of pfBlockerNG configuration and dashboard in `../screenshots/pfblockerng/`.
# pfSense Installation

The process of installing pfSense Community Edition on the Dell OptiPlex hardware.

## Prerequisites

- Dell OptiPlex 7040 with Intel I350-T2 NIC installed in PCIe slot
- USB drive (8 GB or larger) for installer
- Mac for downloading and flashing the installer
- Monitor and USB keyboard for initial setup
- Network cable connecting OptiPlex to ISP modem

## Download and Flash

### Download pfSense CE

Downloaded from `https://www.pfsense.org/download/`:

- Architecture: AMD64 (64-bit)
- Installer: USB Memstick Installer
- File: `pfSense-CE-2.7.X-RELEASE-amd64.img.gz`

### Flash USB Installer

After decompressing the .gz file, flashed to USB drive. On a Windows machine, used Rufus in DD mode. The pfSense `.img` file is a raw disk image, so DD mode is required (ISO mode does not work).

The flashed USB drive shows partition data unrecognizable to Windows or macOS — this is normal. Windows may prompt to format the drive; do not.

### Troubleshooting USB Flashing

Initial Rufus runs failed with "unable to assign drive letter" errors. Resolved by:

1. Cleaning the USB drive with `diskpart`:
2. Closing all File Explorer windows showing the USB
3. Running Rufus as administrator

## BIOS Configuration

Before installation, configured Dell OptiPlex BIOS:

- **Secure Boot:** Disabled (pfSense/FreeBSD does not support Secure Boot)
- **Boot Mode:** UEFI
- **AC Recovery:** Power On (auto-boot after power loss)
- **USB Boot:** Enabled

Pressed F12 at boot to access one-time boot menu and selected the USB drive.

## Installation Process

The Netgate installer ran from USB and prompted for:

1. **License acceptance** — accepted
2. **Install pfSense** — selected (vs. Rescue Shell)
3. **Keymap** — US default
4. **Partitioning** — Auto (ZFS), single disk stripe
5. **Internal drive selection** — selected the OptiPlex's internal SSD
6. **Confirm wipe** — yes

Installer ran for approximately 5 minutes.

### Interface Assignment

After install, the installer prompted for interface assignment:

- 3 NICs detected:
  - `igb0` — first port on Intel I350-T2 (used for WAN)
  - `igb1` — second port on Intel I350-T2 (unused, available for expansion)
  - `em0` — onboard Dell Ethernet (used for LAN)

Mapping rationale: keep onboard NIC for LAN (always present, can't be removed), use add-in card for WAN and future expansion.

The "Should VLANs be set up now?" prompt was answered **no** — VLANs configured later via web GUI.

## Initial Web Configuration

After reboot, the OptiPlex's role transitioned from "machine being installed" to "always-on network appliance." Removed monitor and keyboard.

### Initial Connection

- pfSense LAN port → TP-Link switch port 1
- MacBook → TP-Link switch (via USB-C-to-Ethernet adapter)
- Browsed to `https://192.168.1.1` from MacBook

Browser displayed certificate warning (expected with self-signed cert). Accepted.

Default credentials: `admin` / `pfsense`

### Setup Wizard

Walked through pfSense's first-run wizard:

| Setting | Value |
|---------|-------|
| Hostname | pfsense |
| Domain | home.lab |
| Primary DNS | 1.1.1.1 (Cloudflare) |
| Secondary DNS | 9.9.9.9 (Quad9) |
| Override DNS | Unchecked |
| Time Server | Default pfSense pool |
| Timezone | [your timezone] |
| WAN Type | DHCP |
| Block RFC1918 on WAN | Enabled |
| Block bogons on WAN | Enabled |
| LAN IP | 192.168.1.1/24 |
| Admin Password | Set strong password, saved in password manager |

Applied configuration. pfSense reloaded with new settings.

### Verification

From MacBook on LAN side, confirmed:
ping 192.168.1.1   # Reach pfSense
ping 1.1.1.1       # Reach internet through pfSense
ping google.com    # DNS resolution working
All three successful — basic routing operational.

## Post-Install Hardening

Performed immediately after initial setup:

- **System updates:** System → Update → installed available patches, rebooted
- **Automatic config backups:** Diagnostics → Backup & Restore → enabled
- **Manual config backup:** downloaded XML backup to local password-protected location

## Hardware Configuration Reference

### Network Interface Mapping

| pfSense Interface | Physical Port | Purpose |
|-------------------|---------------|---------|
| WAN | Intel I350-T2 port 1 (igb0) | To ISP modem |
| LAN | Onboard Ethernet (em0) | To TP-Link switch |
| (Unassigned) | Intel I350-T2 port 2 (igb1) | Reserved for future use |

## References

Screenshots of installation and initial configuration available in `../screenshots/pfsense/`.
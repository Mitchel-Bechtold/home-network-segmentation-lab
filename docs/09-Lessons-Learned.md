# Lessons Learned

Honest reflection on what went well, what didn't, and what would be done differently.

## What Went Well

### Design-First Approach

Spending time on the trust-tier model and IP planning before configuration paid off significantly. Subnets aligned with VLAN IDs (192.168.X.0/24 where X is VLAN) made every subsequent step more intuitive.

### Hardware Selection

The used Dell OptiPlex + I350-T2 NIC combination performed well. Total firewall hardware cost (~$130) provides capacity for Suricata IDS and other resource-intensive features without strain.

### Modular Documentation

Building documentation alongside the project (rather than at the end) preserved details that would otherwise be forgotten. Each doc focused on one topic kept individual files manageable.

### Validation Testing

The Kali-based segmentation validation transformed the project from "I configured a network" to "I tested it from a hostile position and proved it works." This is the most valuable artifact for portfolio purposes.

## What Did Not Go Well

### Two GitHub Accounts

Inadvertently created two GitHub accounts at different times, leading to authentication confusion when pushing commits. macOS Keychain silently provided wrong credentials repeatedly.

**Lesson:** Use exactly one GitHub account. If credentials seem wrong, verify by checking which account is logged in via the website.

### Folder Structure Confusion

During Git troubleshooting, accidentally created a nested `home-network-segmentation-lab/home-network-segmentation-lab/` structure with two separate `.git` directories. Required careful manual cleanup to consolidate.

**Lesson:** Always run `pwd` before Git commands to confirm the working directory. VS Code's built-in terminal helps because it always opens in the workspace folder.

### Screenshots Captured Before Sanitization

Took all screenshots at full resolution showing real WAN IP, WiFi passwords, and full MAC addresses. Pushed to public GitHub repo before sanitizing.

**Lesson:** Sanitize before committing, never after. Add a `screenshots/raw/` folder to `.gitignore` and only put sanitized images in the tracked location. Once sensitive data is in Git history, removing it is complex.

The fix required:
1. Making the repo private immediately
2. Rotating WiFi passwords (assume compromised)
3. Sanitizing all images
4. Deleting the GitHub repo and recreating with clean files
5. Reviewing every screenshot before re-pushing

### Omada Controller Approach Wasted Time

Attempted to set up the TP-Link Omada Controller via Docker for AP management before realizing the standalone web UI provides equivalent functionality for a single AP. Multiple hours spent troubleshooting Docker networking, controller adoption, and VLAN-aware discovery.

**Lesson:** Match the tool to the scale. Single-AP deployments don't need centralized controllers. Standalone web UI is simpler and sufficient.

### Initial Firewall Rule Order

Created firewall rules in the wrong order initially (Allow Internet at the top, before block rules). Because pfSense uses first-match-wins evaluation, this defeated the entire policy until reordered.

**Lesson:** "Specific allows above general blocks above catch-alls" is the universal pattern. Verify by checking packet counters — the rule with overwhelming traffic is usually the actual policy enforcer.

## Surprises

### Embedded Device Web UIs Are Often Subnet-Restricted

Many devices (including the TP-Link switch and EAP225) only respond to web management traffic from same-subnet clients. This is a security feature but caught me off guard, leading to the addition of a dedicated management VLAN access port.

This is actually how enterprise networks work — out-of-band management networks are standard practice for exactly this reason.

### Console Discovery Works Differently than Layer 3

The Omada Controller's failure to discover the AP across VLANs was an instructive moment. Layer 2 broadcast-based discovery doesn't traverse VLAN boundaries by design. The same isolation that made segmentation work also broke discovery — a real-world tradeoff.

### Browser/State Caching

Spent significant time debugging "blocked" traffic that was actually allowed because of browser connection reuse and pfSense state table holding established connections. New rules don't kill existing connections — must reset states for changes to fully take effect.

## What I'd Do Differently

### Start with a UPS

Power outages are inevitable. A small UPS (~$70) keeps the firewall running through brief outages and allows graceful shutdown for longer ones. Would prevent state loss and config drift.

### Document While Building, Not After

Writing this documentation a day after completing the work meant rebuilding mental models for things that were obvious during the build. A few notes per session would have made documentation faster and more accurate.

### Sanitize Workflow from Day One

A pre-commit checklist for any image: no public IP, no passwords, no full MACs. Better yet, a `screenshots/raw/` folder gitignored, with a deliberate copy step to `screenshots/` only after redaction.

### Consider WireGuard from the Start

The OpenVPN-vs-WireGuard decision came late. WireGuard's simpler config and better performance would have been worth setting up first, despite NordVPN's lack of native pfSense WireGuard support.

## Skills Developed

Beyond the technical configuration:

- **Network design thinking** — trust tiers, threat modeling, defense in depth
- **Subnetting fluency** — went from book-learned to instinctive
- **Firewall rule logic** — first-match-wins, stateful tracking, rule order
- **Linux/FreeBSD CLI** — pfSense's underlying OS, plus Kali for validation
- **Pen testing methodology** — internal recon as a verification technique
- **Git workflow** — including the troubleshooting that comes with real use
- **Technical documentation** — structuring information for portfolio and reference

## What's Next

This project is foundational for two follow-ups:

- **Project 2: Wazuh SIEM** — log aggregation and detection engineering on top of this network
- **Project 3: Active Directory Lab** — vulnerable AD environment in the Lab VLAN, with attack chains documented and detected by Wazuh

Together, these three projects cover network engineering, defensive operations (blue team), and offensive operations (red team) — a well-rounded portfolio for entry-level cybersecurity roles.
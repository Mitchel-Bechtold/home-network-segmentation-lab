# Firewall Policy

Stateful firewall rules implementing the trust-tier security model.

## Policy Goals

The firewall enforces these high-level rules:

- **Trusted** devices can reach anything (internet, other VLANs, management)
- **IoT/Untrusted** devices can reach internet only
- **Guest** devices can reach internet only, fully isolated
- **Lab** devices have no internet access by default; can resolve DNS only

Each VLAN's policy is implemented through ordered firewall rules on that VLAN's interface in pfSense.

## Aliases

Firewall aliases group IPs/networks/ports for cleaner rule references:

| Alias | Type | Contents | Purpose |
|-------|------|----------|---------|
| RFC1918 | Network(s) | 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 | All private networks |
| pfSense_Web_GUI | Port(s) | 80, 443 | pfSense management ports |
| Mgmt_Net | Network(s) | 192.168.1.0/24 | Management VLAN |

## Rule Logic Reference

pfSense firewall rules follow these principles:

- Rules are evaluated **per interface, on traffic entering that interface**
- Rules are evaluated **top-down**, **first match wins**
- **No matching rule = blocked** (default deny)
- Stateful — return traffic for established connections automatically allowed
- **Apply Changes** must be clicked after saves for rules to take effect
- **State table** holds existing connections — must reset states for new rules to affect ongoing connections

## UNTRUSTED (IoT) Interface Rules

Order critical: specific allows above general blocks above catch-all.

| # | Action | Protocol | Source | Destination | Port | Description |
|---|--------|----------|--------|-------------|------|-------------|
| 1 | Pass | TCP/UDP | UNTRUSTED net | This Firewall | 53 | Allow IoT to query pfSense DNS |
| 2 | Block | any | UNTRUSTED net | This Firewall | * | Block IoT to pfSense management |
| 3 | Block | any | UNTRUSTED net | RFC1918 | * | Block IoT to all private networks |
| 4 | Pass | any | UNTRUSTED net | any | * | Allow IoT to internet only |

### Rule Walkthrough

A DNS query from IoT (UDP port 53) to pfSense matches rule 1 → PASS.

A web request from IoT to pfSense (TCP port 443) does not match rule 1 (wrong port), matches rule 2 → BLOCK.

An attempt by IoT to reach a Trusted device (192.168.10.X) does not match rules 1 or 2 (destination not pfSense), matches rule 3 → BLOCK.

A web request from IoT to google.com does not match rules 1, 2, or 3, matches rule 4 → PASS.

## GUEST Interface Rules

Same pattern as IoT — internet-only with all private network access blocked:

| # | Action | Protocol | Source | Destination | Port | Description |
|---|--------|----------|--------|-------------|------|-------------|
| 1 | Pass | TCP/UDP | GUEST net | This Firewall | 53 | Allow Guest DNS |
| 2 | Block | any | GUEST net | This Firewall | * | Block Guest to pfSense |
| 3 | Block | any | GUEST net | RFC1918 | * | Block Guest to private networks |
| 4 | Pass | any | GUEST net | any | * | Allow Guest to internet |

## LAB Interface Rules

Default-deny policy. Reject (not Block) for internet — apps fail fast instead of timing out.

| # | Action | Protocol | Source | Destination | Port | Description |
|---|--------|----------|--------|-------------|------|-------------|
| 1 | Pass | TCP/UDP | LAB net | This Firewall | 53 | Allow Lab DNS resolution |
| 2 | Block | any | LAB net | This Firewall | * | Block Lab to pfSense |
| 3 | Block | any | LAB net | RFC1918 | * | Block Lab to private networks |
| 4 | Reject | any | LAB net | any | * | Reject Lab internet (default deny) |

When specific Lab projects need internet (e.g., updating Kali), a temporary Pass rule is added above rule 4.

## TRUSTED Interface Rules

Permissive — Trusted devices can reach anything:

| # | Action | Protocol | Source | Destination | Port | Description |
|---|--------|----------|--------|-------------|------|-------------|
| 1 | Pass | TCP | TRUSTED net | This Firewall | pfSense_Web_GUI | Trusted to manage pfSense |
| 2 | Pass | TCP/UDP | TRUSTED net | This Firewall | 53 | Trusted DNS |
| 3 | Pass | any | TRUSTED net | any | * | Allow Trusted everywhere |

This treats Trusted devices as well-behaved. A more restrictive policy could be added later (block Trusted → other VLANs except specific exceptions), but for a daily-use network, permissive Trusted is reasonable.

## Logging

Block rules on UNTRUSTED, GUEST, and LAB have logging enabled (Extra Options → Log packets that are handled by this rule). This provides visibility into denied attempts.

Pass rules are not logged — too high volume for normal traffic.

The default deny logging (Status → System Logs → Settings) is enabled for visibility into anything blocked by the implicit catch-all.

## Validation

Firewall policy validated through internal penetration testing — see [docs/08-segmentation-validation.md](08-segmentation-validation.md).

## Troubleshooting Notes

### Rule Order Inverted

Initial IoT rules were inverted with "Allow Internet" at the top. Because of "first match wins," this rule matched all traffic before the block rules could be evaluated, defeating the entire policy.

Diagnosis: rule packet counters showed massive traffic on Allow Internet (367 MiB) while block rules showed 0 packets.

Resolution: reordered rules so specific allows are at top, blocks in middle, catch-all internet allow at bottom.

### Rules Don't Take Effect Immediately

Existing connections continue to work after a new block rule is added. pfSense's state table holds active connections, which aren't subject to new rules.

Resolution: Diagnostics → States → Reset States → check box → Reset. Forces all connections to re-evaluate against current rules.

### Browser Cache Misled Testing

After applying block rules to IoT, browser still loaded pfSense web UI from IoT. Cause was browser caching the previous session, not the firewall failing.

Resolution: force-close browser, wait 30 seconds, reopen — connection re-attempted from scratch and was correctly blocked.

## References

Screenshots of all firewall rules in `../screenshots/rules/`.
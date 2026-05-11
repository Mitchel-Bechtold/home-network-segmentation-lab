# Segmentation Validation

Internal penetration testing to validate the network segmentation policy. Conducted using Kali Linux on the LAB VLAN, simulating a hostile insider or compromised device.

## Methodology

A Kali Linux VM was deployed on the LAB VLAN (192.168.40.0/24) and used as an attacker workstation. Standard reconnaissance and scanning tools were employed to attempt to enumerate, reach, or fingerprint resources on other VLANs.

The hypothesis: segmentation policy should prevent any cross-VLAN reachability from the hostile position.

### Tools Used

- `nmap` — network discovery and port scanning
- `arp-scan` — Layer 2 reconnaissance
- `dig` — DNS query testing
- `curl` — HTTP connectivity testing
- `ping` — ICMP reachability

### Test Environment

| Component | Configuration |
|-----------|---------------|
| Attacker host | Kali Linux 2025.X VM in VMware Fusion |
| Network mode | Bridged to physical Ethernet |
| Physical port | Switch port 5 (LAB VLAN access) |
| Attacker IP | 192.168.40.X (DHCP from pfSense) |

## Test Results

### Test 1: Lab VLAN Internal Sweep (Control)

- sudo nmap -sn 192.168.40.0/24

**Result:** 3 hosts discovered:
- 192.168.40.1 (pfSense gateway, identified as Dell hardware via MAC OUI)
- 192.168.40.X (host Mac, Realtek USB Ethernet adapter)
- 192.168.40.X (Kali attacker VM)

**Significance:** Control test confirming scanning tools function correctly. Layer 2-adjacent hosts in the same VLAN are discoverable via ARP, as expected.

Scan completed in 1.91 seconds.

### Test 2: Trusted VLAN Sweep

- sudo nmap -sn 192.168.10.0/24

**Result:** `Nmap done: 256 IP addresses (0 hosts up)`

**Significance:** From hostile position on Lab, no hosts on Trusted VLAN are discoverable. Despite known active devices on Trusted (laptops, phones), nmap finds nothing.

Scan completed in approximately 50 seconds — slow because every probe times out (firewall silently drops them) rather than receiving immediate ARP-failure responses (which would indicate same-VLAN scanning of empty addresses).

### Test 3: IoT VLAN Sweep

- sudo nmap -sn 192.168.20.0/24

**Result:** 0 hosts up.

**Significance:** IoT VLAN unreachable from Lab.

### Test 4: Guest VLAN Sweep

- sudo nmap -sn 192.168.20.0/24

**Result:** 0 hosts up.

**Significance:** IoT VLAN unreachable from Lab.

### Test 4: Guest VLAN Sweep

- sudo nmap -sn 192.168.30.0/24

**Result:** 0 hosts up.

**Significance:** Guest VLAN unreachable from Lab.

### Test 5: Management VLAN Sweep

- sudo nmap -sn 192.168.1.0/24

**Result:** 0 hosts up.

**Significance:** Management network completely isolated from Lab. pfSense management interface, switch management interface, and AP management interface are all unreachable from a hostile Lab position.

### Test 6: Layer 2 Reconnaissance

- sudo arp-scan --interface=eth0 --localnet

**Result:** 2 responses — only Lab VLAN devices (pfSense gateway, host Mac).

**Significance:** ARP traffic does not cross VLAN boundaries. Layer 2 broadcast isolation working as designed. An attacker cannot use ARP-based reconnaissance to discover devices on other VLANs from this position.

### Test 7: Targeted Port Scan Against pfSense

- sudo nmap -p 22,80,443 192.168.1.1 192.168.10.1 192.168.20.1 192.168.30.1

**Result:** All ports filtered or hosts unreachable.

**Significance:** pfSense management interfaces are not exposed to the Lab VLAN. Web UI cannot be reached, SSH cannot be reached. Even attempting to identify pfSense by fingerprinting fails.

For comparison, the same scan from Trusted VLAN would show port 443 open and pfSense web UI accessible. Same firewall, same machine — visibility differs based on source VLAN.

### Test 8: DNS Filtering Validation

- dig @192.168.40.1 doubleclick.net

**Result:** Returns 0.0.0.0 (sinkhole)

**Significance:** DNS-layer threat filtering applies to traffic from Lab VLAN. Even attacker-controlled hosts attempting to resolve known-bad domains receive sinkhole responses.

### Test 9: Internet Egress

- ping -c 3 8.8.8.8 curl -I --max-time 5 https://google.com

**Result:** Both fail — destination unreachable, connection refused.

**Significance:** LAB VLAN's default-deny internet policy is enforced. Despite DNS resolution working (allowed for VM functionality), TCP/UDP traffic to the internet is rejected.

## Firewall Log Evidence

During testing, the pfSense firewall log was monitored (Status → System Logs → Firewall, filtered by Kali's source IP).

Hundreds of Block entries were logged in real time, showing:
- Source: 192.168.40.X (Kali)
- Destination: 192.168.1.X, 192.168.10.X, 192.168.20.X (target VLANs)
- Action: Block (red X)
- Rule: matching the per-VLAN block rules

This provides real-time evidence that segmentation is actively enforced, not just configured.

## Summary

| Test | Expected | Actual | Pass/Fail |
|------|----------|--------|-----------|
| Lab internal sweep | 3 hosts | 3 hosts | ✓ Pass |
| Trusted sweep | 0 hosts | 0 hosts | ✓ Pass |
| IoT sweep | 0 hosts | 0 hosts | ✓ Pass |
| Guest sweep | 0 hosts | 0 hosts | ✓ Pass |
| Management sweep | 0 hosts | 0 hosts | ✓ Pass |
| ARP scan (L2) | Lab only | Lab only | ✓ Pass |
| pfSense port scan | Filtered | Filtered | ✓ Pass |
| DNS filtering | Sinkhole | Sinkhole | ✓ Pass |
| Internet egress | Blocked | Blocked | ✓ Pass |

All tests passed. Network segmentation operates as designed.

## Conclusion

From a hostile position on the Lab VLAN, no resources on Trusted, IoT, Guest, or Management VLANs are reachable, scannable, or enumerable. Defense in depth is enforced at both Layer 2 (VLAN broadcast isolation) and Layer 3 (stateful firewall policy).

A compromised device or hostile insider on Lab cannot:
- Discover hosts on other VLANs
- Reach pfSense management interfaces
- Access switch or AP management
- Pivot to Trusted, IoT, or Guest networks
- Resolve malicious domains via the network's DNS resolver

This validates the design goal of containing threats within the Lab segment and protecting more-trusted segments from compromise.

## References

Screenshots of all test outputs and firewall log evidence in `../screenshots/kali-validation/`.



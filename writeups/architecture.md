# SOC Homelab Architecture

---

## 1. Scope Statement

This SOC homelab simulates a smaller-end industrial equipment manufacturer. The company is modeled loosely after CAT (Caterpillar), since I have been working on one of their "sponsored" projects this summer with CERIAS. The business is broken up into two spheres: corporate and operations. Corporate will deal with the usual enterprise security concerns across both the physical domain (CCTV, keypads, and doorlocks) and the digital domain (Active Directory, network monitoring, endpoint protection). Operations will deal with many of the same physical operations but on the digital-end inherit more OT-specific concerns: segmentation from the corporate network, industrial protocol monitoring, PLC register access logging, and anomaly detection on workstation traffic.

---

## 2. Threat Model

The company's threat model acknowledges three potentially malicious actors: ransomware gang, nation-state, and insider threat.

Ransomware gangs want money, and obtain it by holding company services and data hostage until they get it. Ransomware gangs would exercise their malicious operations through spearphishing into the corporate domain, lateral movement to OT, and encryption of both IT systems and historian data.

Nation-state actors might want to physically sabotage company operations, as seen in Stuxnet's PLC compromise. They could do this through supply chain compromise or spearphishing leading to PLC logic manipulation.

Finally, insider threats come from within the company and can be a result of employee dissatisfaction. In the case of an insider threat, more intimate vulnerabilities would have to be guarded like unsecured networks, physical USB ports on equipment, or even open laptops left unguarded in the office.

Each threat actor corresponds to a dedicated attack campaign executed in Phase 3.

---

## 3. Network Segmentation

| Segment | What's in it | Trust Level |
|---------|-------------|-------------|
| Corporate/IT | Active Directory (AD), workstations, file shares, email | Medium: Authenticated Users |
| SOC/Monitoring | SIEM, log collectors, dashboards | High: Security Team |
| OT/ICS | PLCs, HMIs, engineering workstations | High: Security Team |
| Attacker | Kali, attack tools | None |
| Malware Analysis | Sandbox VMs, RE tools | Zero (Isolated) |

### Design Rules

1. Every segment gets its own subnet
2. A firewall sits between every segment
3. Default deny between segments
4. The monitoring segment sees everything but is reachable by nothing
5. The malware analysis segment routes to nothing

### Libvirt Virtual Networks

| Virtual Network | Subnet | Internet Access |
|----------------|--------|-----------------|
| corp-net | 10.1.0.0/24 | Yes (filtered through pfSense) |
| soc-net | 10.2.0.0/24 | Limited (updates and threat feeds only) |
| ot-net | 10.3.0.0/24 | No |
| attacker-net | 10.4.0.0/24 | Temporary (during campaigns only) |
| malware-net | 10.5.0.0/24 | Absolutely not |

---

## 4. IP Allocation Table

| VM | Subnet | IP | OS | Role | RAM | Disk |
|----|--------|----|----|------|-----|------|
| pfSense | All segments | 10.1.0.1, 10.2.0.1, 10.3.0.1, 10.4.0.1, 10.5.0.1 | FreeBSD | Firewall/router between all segments | 1GB | 10GB |
| Security Onion | SOC/Monitoring (10.2.0.0/24) | 10.2.0.10 | Linux | SIEM, IDS, network monitoring | 8GB | 200GB |
| Windows Server 2019 | Corporate/IT (10.1.0.0/24) | 10.1.0.10 | Windows | Domain controller, AD, DNS | 4GB | 40GB |
| Windows 10 | Corporate/IT (10.1.0.0/24) | 10.1.0.100 | Windows | Domain-joined target workstation | 4GB | 40GB |
| OpenPLC | OT/ICS (10.3.0.0/24) | 10.3.0.10 | Linux | Simulated PLC, Modbus TCP on port 502 | 1GB | 10GB |
| ScadaBR | OT/ICS (10.3.0.0/24) | 10.3.0.20 | Linux | HMI, talks Modbus to OpenPLC | 2GB | 15GB |
| Kali Linux | Attacker (10.4.0.0/24) | 10.4.0.100 | Kali Linux | Attack platform | 2GB | 40GB |
| FlareVM | Malware Analysis (10.5.0.0/24) | 10.5.0.100 | Windows | Windows malware analysis | 4GB | 60GB |
| REMnux | Malware Analysis (10.5.0.0/24) | 10.5.0.101 | Linux | Linux malware analysis | 2GB | 20GB |
| MISP | SOC/Monitoring (10.2.0.0/24) | 10.2.0.20 | Linux | Threat intel platform | 4GB | 20GB |

---

## 5. Security Onion Placement

| Interface | Type | Libvirt Network | Has IP? | Purpose |
|-----------|------|-----------------|---------|---------|
| NIC 1 | Management | soc-net | Yes (10.2.0.10) | Access web UI, manage the box |
| NIC 2 | Monitoring | corp-net | No | Passive capture of IT traffic |
| NIC 3 | Monitoring | ot-net | No | Passive capture of OT traffic |

---

## 6. Traffic Flows and Firewall Rules

### CORP-NET interface

| Order | Action | Source | Dest | Port | Why |
|-------|--------|--------|------|------|-----|
| 1 | PASS | 10.1.0.0/24 | 10.1.0.10 | 53/UDP | DNS to domain controller |
| 2 | PASS | 10.1.0.0/24 | 10.2.0.10 | 514/UDP | Syslog to Security Onion |
| 3 | BLOCK | 10.1.0.0/24 | 10.3.0.0/24 | * | Corporate never reaches OT |
| 4 | BLOCK | 10.1.0.0/24 | * | * | Default deny |

### OT-NET interface

| Order | Action | Source | Dest | Port | Why |
|-------|--------|--------|------|------|-----|
| 1 | PASS | 10.3.0.0/24 | 10.2.0.10 | 514/UDP | Syslog to Security Onion |
| 2 | BLOCK | 10.3.0.0/24 | 10.1.0.0/24 | * | OT never reaches corporate |
| 3 | BLOCK | 10.3.0.0/24 | * | * | Default deny |

### SOC-NET interface

| Order | Action | Source | Dest | Port | Why |
|-------|--------|--------|------|------|-----|
| 1 | PASS | 10.2.0.0/24 | 10.1.0.0/24 | 22, 3389 | IR access to corporate hosts |
| 2 | PASS | 10.2.0.0/24 | * | 443, 80 | Updates and threat feeds |
| 3 | BLOCK | 10.2.0.0/24 | * | * | Default deny |

### ATTACKER-NET interface

| Order | Action | Source | Dest | Port | Why |
|-------|--------|--------|------|------|-----|
| 1 | BLOCK | 10.4.0.0/24 | * | * | Default deny; campaign rules added temporarily |

### MALWARE-NET

No rules. No gateway. No traffic. Full isolation.

---

## 7. ICS/OT Segment Design

The OT segment simulates a small manufacturing floor with a single PLC controlling a basic industrial process. OpenPLC (10.3.0.10) runs a ladder logic program that reads input values and sets output states in a continuous scan cycle; for example, monitoring a simulated tank level sensor and controlling a fill valve. The PLC exposes its registers over Modbus TCP on port 502.

ScadaBR (10.3.0.20) acts as the HMI, polling OpenPLC's holding registers (function code 03) on a fixed interval (every 1 second). The ScadaBR web dashboard displays process values in real-time: gauges, indicators, and status lights that an operator would monitor during normal production.

This traffic is entirely intra-subnet. Both devices sit on ot-net (10.3.0.0/24) and communicate directly through the virtual switch without passing through pfSense. pfSense's ot-net interface only handles traffic leaving or entering the OT segment. Security Onion's sniffing NIC (NIC 3) on ot-net captures all Modbus traffic passively.

### Normal Traffic Baseline

| Attribute | Expected Value |
|-----------|---------------|
| Source IP | 10.3.0.20 (ScadaBR only) |
| Destination IP | 10.3.0.10 (OpenPLC only) |
| Port | 502/TCP |
| Function Code | 03 (Read Holding Registers) |
| Interval | Every 1 second, consistent |
| Direction | ScadaBR → OpenPLC (request), OpenPLC → ScadaBR (response) |

### Suspicious Traffic Indicators

| Indicator | Why it's suspicious |
|-----------|-------------------|
| New source IP on port 502 | Only ScadaBR should talk to the PLC; any other IP is unauthorized |
| Write function codes (05, 06, 15, 16) | Normal operations are read-only polling; writes indicate someone is changing the process |
| Irregular polling interval | Sudden burst of requests or long gaps break the deterministic pattern |
| Connection from corp-net or attacker-net | Purdue Model violation; IT should never directly reach OT devices |
| Modbus exception responses | The PLC is rejecting commands it doesn't understand, indicating scanning or fuzzing |

---

## 8. Malware Analysis Isolation Design

The malware analysis segment (malware-net, 10.5.0.0/24) is fully isolated from all other segments and the internet. In libvirt, this network is configured as an "isolated" type virtual network: no NAT, no default route, no IP forwarding enabled on the host for this bridge. pfSense does not have a NIC on malware-net. There is no gateway address, meaning VMs on this segment have no path to any other network.

FlareVM (10.5.0.100) and REMnux (10.5.0.101) can communicate with each other on malware-net; this is intentional. REMnux runs fakenet-ng, which simulates DNS, HTTP, and HTTPS services locally so that malware samples believe they have internet connectivity while all traffic stays contained within the isolated segment.

### File Transfer Procedure

Samples are transferred into the analysis VMs using the virt-manager console's copy-paste functionality or by mounting a shared folder on the host that is read-only from the VM side. Samples are never transferred over the network. Analysis reports, YARA rules, and screenshots are transferred out the same way: through the virt-manager console or shared folder, never over a network connection.

### Isolation Verification Test

Before analyzing any samples, the following tests must be performed from both FlareVM and REMnux and all must fail:

| Test | Command | Expected Result |
|------|---------|-----------------|
| Ping host OS | `ping 10.1.0.1` (or host IP) | No response, request timeout |
| Ping internet | `ping 8.8.8.8` | No response, network unreachable |
| DNS resolution | `nslookup google.com` | Failure, no DNS server reachable |
| Reach another segment | `ping 10.2.0.10` | No response, network unreachable |
| Reach gateway | `ping 10.5.0.1` | No response (no gateway exists) |

Results of these tests are documented in the build log before any malware analysis begins.

---

## 9. VM Resource Budget

Hardware constraints: 24GB DDR4 RAM, 1TB NVMe SSD. Target: never exceed 16GB total VM RAM per phase, leaving 8GB for Ubuntu host and KVM/libvirt overhead.

| Phase | VMs Running | Total RAM | Total Disk | Notes |
|-------|------------|-----------|------------|-------|
| 1-2 (Infra + Detection) | pfSense (1GB), SecOnion (8GB), WinServer (4GB), Win10 (4GB) | 17GB | 330GB | Tight — reduce Win10 to 3GB or stagger AD setup and target separately |
| 3a (Ransomware campaign) | pfSense (1GB), SecOnion (8GB), WinServer (4GB), Kali (2GB) | 15GB | 290GB | Win10 shut down if needed, target WinServer directly |
| 3b (ICS campaign) | pfSense (1GB), SecOnion (8GB), OpenPLC (1GB), ScadaBR (2GB), Kali (2GB) | 14GB | 275GB | Comfortable — AD and Win10 shut down |
| 3c (Insider campaign) | pfSense (1GB), SecOnion (8GB), WinServer (4GB), Win10 (4GB) | 17GB | 330GB | Same squeeze as Phase 1-2, stagger if needed |
| 4 (Threat Intel) | pfSense (1GB), SecOnion (8GB), MISP (4GB) | 13GB | 230GB | Light phase |
| 5 (Malware Analysis) | FlareVM (4GB), REMnux (2GB) | 6GB | 80GB | SecOnion and pfSense OFF — very light |

### Disk Budget

Total VM disk allocation across all VMs: 455GB. With snapshots averaging 10GB each and an estimated 15-20 snapshots over the project lifecycle, snapshot storage adds approximately 150-200GB. Total estimated disk usage: ~650GB of 1TB, leaving comfortable headroom.

### Mitigation for Phase 1-2 and 3c RAM squeeze

If 17GB causes host instability, reduce Win10 to 3GB (16GB total) or build and configure WinServer first, snapshot it, shut it down, then build and configure Win10. Only run both simultaneously when testing AD-joined detection rules.

---

## 10. Snapshot Strategy

### Naming Convention

```
[vm-name]_[phase]_[state]_[date]
```

Examples:

| Snapshot Name | When it's taken |
|--------------|-----------------|
| `securityonion_p1_baseline_20260804` | After Security Onion is fully configured and ingesting logs |
| `winserver_p1_ad-configured_20260806` | After AD, DNS, and Group Policy are set up |
| `win10_p1_domain-joined_20260807` | After Win10 is joined to the domain with Sysmon installed |
| `win10_p3a_pre-ransomware_20260818` | Clean state before ransomware campaign begins |
| `openplc_p3b_pre-ics-attack_20260822` | Clean state before ICS attack campaign |
| `flarevm_p5_clean_20260828` | Clean state before every malware sample — revert after each analysis |

### When to Snapshot

- After completing initial VM setup and configuration (golden baseline)
- Before every attack campaign (clean state to revert to)
- Before every malware sample (critical — never accumulate state in analysis VMs)
- Before major configuration changes (new detection rules, network changes)

### When to Prune

- After completing a phase, delete intermediate snapshots and keep only the golden baseline
- Keep one golden baseline per VM at all times
- Keep one pre-campaign snapshot per campaign until the IR report is written
- Monitor disk usage weekly — run `sudo du -sh /var/lib/libvirt/images/` to check

### Recovery Procedure

If a VM is corrupted or a campaign leaves artifacts that can't be cleaned, revert to the most recent clean snapshot. Do not attempt to manually fix a compromised target VM — revert and re-run. The snapshot is always cheaper than debugging.


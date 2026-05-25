# 🔴 Red Team Attack Simulation & Detection with Splunk

![Lab Status](https://img.shields.io/badge/Lab-Complete-brightgreen)
![Platform](https://img.shields.io/badge/Platform-VirtualBox-blue)
![SIEM](https://img.shields.io/badge/SIEM-Splunk-orange)
![Attacker](https://img.shields.io/badge/Attacker-Kali%20Linux-red)

> Simulating real-world cyber attacks from Kali Linux against a Windows 10 target and detecting every move using Splunk Enterprise.

---

## 📋 Project Overview

This project demonstrates a complete **red team → blue team** detection pipeline built in a home lab environment. Attacks are launched from Kali Linux, logged by Windows, forwarded to Splunk, and visualized on a custom detection dashboard.

**Total Events Collected: 216,872**

---

## 🏗️ Lab Architecture

```
┌─────────────────┐         ┌─────────────────┐
│   Kali Linux    │ ──────▶ │   Windows 10    │
│  192.168.56.105 │  Attack │  192.168.56.104 │
│    (Attacker)   │         │    (Target)     │
└─────────────────┘         └────────┬────────┘
                                     │ Logs
                                     ▼
                            ┌─────────────────┐
                            │     Splunk      │
                            │  localhost:8000 │
                            │    (Detector)   │
                            └─────────────────┘
```

**Network:** Oracle VirtualBox — Host-Only Network (192.168.56.0/24)

---

## 🛠️ Tools Used

| Tool | Version | Purpose |
|------|---------|---------|
| Kali Linux | 2024.x | Attack platform |
| Windows 10 | 22H2 | Target machine |
| Splunk Enterprise | 10.2.3 | SIEM and detection |
| Splunk Universal Forwarder | Latest | Log forwarding |
| Sysmon | v15.x | Enhanced endpoint logging |
| Nmap | 7.99 | Port scanning |
| enum4linux | 0.9.1 | SMB enumeration |
| Hydra | v9.6 | Brute force |
| Metasploit | v6.4 | Vulnerability scanning |

---

## ⚔️ Attacks Simulated

### 1. Reconnaissance — Nmap Port Scan
```bash
nmap -sV -A -T4 192.168.56.104
```
**Findings:** Open ports 135, 139, 445, 8000, 8089 | OS: Windows 10 22H2

**Splunk Detection:** Spike in Event ID 5156/5157 (1,116 events)

---

### 2. Enumeration — SMB (enum4linux)
```bash
enum4linux -a 192.168.56.104
```
**Findings:** Domain: WORKGROUP | Usernames leaked: administrator, guest, krbtgt

**Splunk Detection:** SMB session attempts in Security log

---

### 3. Brute Force — Hydra
```bash
hydra -l Administrator -P /usr/share/wordlists/rockyou.txt rdp://192.168.56.104 -t 4 -V
```
**Findings:** Multiple failed authentication attempts generated

**Splunk Detection:** Event ID 4625 spikes (Failed Logon)

---

### 4. Vulnerability Scan — Metasploit EternalBlue
```bash
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS 192.168.56.104
run
```
**Findings:** Target NOT vulnerable to MS17-010 (patched)

**Splunk Detection:** SMB probe visible in network connection logs

---

## 📊 Splunk Dashboard

**Dashboard Name:** Red Team Attack Detection Lab

### Panels

| Panel | SPL Query | Visualization |
|-------|-----------|---------------|
| Attack Timeline | `index=main host=DESKTOP-EVMUNVQ \| timechart count by EventCode` | Line Chart |
| Failed Logons | `index=main EventCode=4625 \| timechart count` | Column Chart |
| Top Event Codes | `index=main \| stats count by EventCode \| sort -count \| head 10` | Bar Chart |
| Network Connections | `index=main EventCode=5156 \| stats count by Destination_Address Destination_Port \| sort -count \| head 10` | Table |
| Process Creation | `index=main EventCode=4688 \| stats count by Creator_Process_Name \| sort -count \| head 10` | Bar Chart |
| Total Events | `index=main host=DESKTOP-EVMUNVQ \| stats count` | Single Value |

---

## 🔍 Key Event IDs Reference

| Event ID | Meaning | Attack Relevance |
|----------|---------|-----------------|
| 4624 | Successful Logon | Tracks access |
| 4625 | Failed Logon | Brute force detection |
| 4688 | Process Created | Attack tool execution |
| 4689 | Process Terminated | Attack tool activity |
| 4703 | Token Rights Adjusted | Privilege activity |
| 5156 | Network Connection Allowed | Port scan detection |
| 5157 | Network Connection Blocked | Firewall activity |

---

## 📁 Repository Structure

```
red-team-splunk-lab/
│
├── configs/
│   ├── inputs.conf          # Splunk forwarder input config
│   ├── outputs.conf         # Splunk forwarder output config
│   └── sysmonconfig.xml     # Sysmon configuration
│
├── dashboards/
│   └── red_team_detection_lab.xml   # Splunk dashboard export
│
├── screenshots/
│   ├── Attack_1_Port_Scan.png
│   ├── Attack_2_SMB_Enumeration.png
│   ├── Attack_3_EternalBlue.png
│   ├── Splunk_Detection_BruteForce.png
│   ├── Splunk_EventCode_Summary.png
│   └── Dashboard_Full_View.pdf
│
└── README.md
```

---

## 📈 Results Summary

```
Total Events Collected    →  216,872
Event Types Detected      →  106 unique Event Codes
Network Events (5156)     →  1,116
Process Events (4688)     →  26,750
Failed Logons (4625)      →  4
Successful Logons (4624)  →  322
```

---

## 🧠 Key Learnings

- Every attack leaves a trace — Nmap, enum4linux, and Hydra all generated distinct, detectable patterns
- Sysmon dramatically improves endpoint visibility beyond standard Windows logging
- SPL queries can surface attack patterns from hundreds of thousands of events in seconds
- Network configuration in a virtual lab environment requires careful attention to subnets and adapters
- Red team thinking sharpens blue team detection — understanding the attack makes the detection rule better

---

## 🚀 What's Next

- [ ] Add Metasploitable2 as a second target
- [ ] Build Splunk alerts that auto-trigger on brute force patterns
- [ ] Simulate post-exploitation activity (lateral movement, persistence)
- [ ] Document threat hunting techniques using this dataset
- [ ] Add MITRE ATT&CK mapping for each attack

---

## 📖 Full Write-Up

Read the complete technical article on Medium:
**[How I Simulated Real Cyber Attacks and Detected Them Using Splunk](https://medium.com/@shivamsinghsengar)**

---

## 👤 Author

**Shivam Singh Sengar (S-aghori)**

- GitHub: [github.com/S-aghori](https://github.com/S-aghori)
- LinkedIn: [Shivam Singh Sengar](https://linkedin.com/in/shivamsinghsengar)

---

> ⚠️ **Disclaimer:** This project was conducted entirely in an isolated home lab environment for educational purposes. All attacks were performed against virtual machines I own and control.

---

⭐ If this project helped you, consider giving it a star!

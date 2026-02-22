# Port-Scan-Detection-LAB

# 🛡️ Port Scan Detection Engineering Lab

> A hands-on cybersecurity lab for detecting and analyzing port scan attacks using **Kali Linux**, **Nmap**, and **Splunk**.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Phase 1: Lab Setup](#phase-1-lab-setup)
- [Phase 2: Running Nmap Scans](#phase-2-running-nmap-scans)
- [Phase 3: Splunk Detection Dashboards](#phase-3-splunk-detection-dashboards)
- [Detection Logic Explained](#detection-logic-explained)
- [Learning Outcomes](#learning-outcomes)

---

## 🔍 Overview

This lab simulates real-world port scanning attacks and demonstrates how to detect them using log analysis in Splunk. You'll learn how attackers use Nmap to enumerate open ports and services, and how defenders can identify this behavior through firewall logs.

---

## 🏗️ Architecture

| Machine | OS | Role |
|---|---|---|
| 🐉 Attacker | Kali Linux | Runs Nmap scans |
| 🖥️ Target / Victim | Ubuntu / Windows | Receives scans, generates logs |
| 📊 Log Analyzer | Splunk Server | Ingests and analyzes logs |

> All machines should be on the same local/virtual network for the lab to function correctly.

---

## ✅ Prerequisites

- VirtualBox or VMware (for running multiple VMs)
- Kali Linux ISO
- Ubuntu Server/Desktop ISO
- Splunk Free (or Enterprise Trial) installed on a dedicated VM or the Ubuntu machine
- Basic familiarity with Linux CLI

---

## Phase 1: Lab Setup

### 🖥️ Configure the Target Machine (Ubuntu)

**1. Check and enable UFW Firewall:**

```bash
sudo ufw status
```

If inactive:

```bash
sudo ufw enable
```

**2. Enable UFW logging:**

```bash
sudo ufw logging on
```

Logs will now be written to:

```
/var/log/ufw.log
```

**3. Verify logs are flowing:**

```bash
sudo tail -f /var/log/ufw.log
```



---

## Phase 2: Running Nmap Scans

### 🐉 On the Kali (Attacker) Machine

**First, identify your target IP:**

```bash
ip a
```

### Scan Types

Run each scan one by one and observe the UFW logs on the Ubuntu terminal simultaneously.

| Scan | Command | Description |
|---|---|---|
| TCP Connect Scan | `nmap -sT <target-ip>` | Full TCP handshake — most detectable |
| SYN (Stealth) Scan | `sudo nmap -sS <target-ip>` | Half-open scan, faster and stealthier |
| FIN Scan | `sudo nmap -sF <target-ip>` | Sends FIN packets to probe closed ports |
| Xmas Scan | `sudo nmap -sX <target-ip>` | Sets FIN, PSH, URG flags simultaneously |
| Aggressive Scan | `sudo nmap -A <target-ip>` | OS detection, version detection, scripts |
| Full Port Scan | `sudo nmap -p- <target-ip>` | Scans all 65535 ports |

**Example:**

```bash
nmap -sT 192.168.1.20
sudo nmap -sS 192.168.1.20
sudo nmap -sF 192.168.1.20
sudo nmap -sX 192.168.1.20
sudo nmap -A 192.168.1.20
sudo nmap -p- 192.168.1.20
```

> 💡 **Tip:** Keep `sudo tail -f /var/log/ufw.log` running on the Ubuntu machine while you execute scans from Kali. You'll see log entries appear in real time.

---

## Phase 3: Splunk Detection Dashboards

Create a new Splunk dashboard with the following three panels.

<img width="1919" height="889" alt="Screenshot 2026-02-22 185745" src="https://github.com/user-attachments/assets/1afe0429-9d98-4d3f-9dd1-6eda7052d1ae" />

---

### Panel 1 — 🔎 Port Scan Detection

Identifies source IPs that have probed more than 10 unique destination ports — a strong indicator of scanning activity.

```spl
index=*
| stats dc(DPT) as unique_ports by SRC
| where unique_ports > 10
```
<img width="942" height="193" alt="image" src="https://github.com/user-attachments/assets/2f413cd0-0173-49c9-94e0-eec85062531b" />

**What it detects:** Any host scanning more than 10 distinct ports within the indexed time range.

---

### Panel 2 — ⚡ High Frequency Connection Attempts

Identifies IPs generating a high volume of UFW-blocked attempts per minute — useful for catching aggressive or noisy scanners.

```spl
index=* "UFW"
| rex "SRC=(?<src_ip>\S+)"
| bucket _time span=1m
| stats count by src_ip, _time
| sort - count
```
<img width="1874" height="337" alt="image" src="https://github.com/user-attachments/assets/f3929c89-02be-42b9-90aa-aef4212671b4" />

**What it detects:** Bursts of blocked connection attempts grouped by minute, sorted by frequency.

---

### Panel 3 — 🚨 SYN Scan Detection

Flags source IPs sending SYN packets, a hallmark of stealth SYN scanning (`nmap -sS`).

```spl
index=* "SYN"
| rex "SRC=(?<src_ip>\S+)"
| stats count by src_ip
| sort - count
```
<img width="933" height="333" alt="image" src="https://github.com/user-attachments/assets/5bde1f73-7d3e-48a2-a61b-a0b7c91e2715" />

**What it detects:** Hosts sending SYN-only packets without completing the TCP handshake — characteristic of SYN/stealth scans.

---

## 🧠 Detection Logic Explained

| Attack Technique | Nmap Flag | Log Indicator | Splunk Panel |
|---|---|---|---|
| TCP Connect Scan | `-sT` | Multiple ALLOW/BLOCK entries | Panel 1, 2 |
| SYN Stealth Scan | `-sS` | `SYN` flag in UFW logs | Panel 3 |
| FIN / Xmas Scan | `-sF`, `-sX` | Unusual TCP flag combos | Panel 1, 2 |
| Aggressive Scan | `-A` | Multiple services probed | Panel 1, 2 |
| Full Port Scan | `-p-` | Very high `unique_ports` count | Panel 1 |

---

## 🎯 Learning Outcomes

After completing this lab, you will be able to:

- Understand how different Nmap scan types work at the packet level
- Configure UFW firewall logging on Linux systems
- Ingest and parse firewall logs in Splunk
- Write SPL (Search Processing Language) queries to detect scanning behavior
- Build a basic threat detection dashboard in Splunk
- Recognize the difference between noisy vs. stealthy scan techniques

---


## ⚠️ Disclaimer

This lab is for **educational purposes only**. Only run port scans against machines you own or have explicit written permission to test. Unauthorized scanning is illegal.

---


## 👤 Author

**Akish Raj.A**
- LinkedIn: https://www.linkedin.com/in/akish-raj/


---

## 🏷 Tags

`Cybersecurity` `SOC` `Splunk` `SIEM` `Nmap` `portscan` 
`LogAnalysis` `KaliLinux` `EthicalHacking` `BlueTeam`

## 📜 License

MIT License — see [LICENSE](LICENSE) for details.

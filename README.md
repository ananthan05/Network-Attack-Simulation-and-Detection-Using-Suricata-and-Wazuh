# Simulating and Detecting LAN Attacks Using Wazuh and Suricata

This project demonstrates how to simulate and detect 14 different LAN-based network attacks using an isolated virtual lab environment. It combines offensive (attacks) and defensive (detection) techniques using powerful open-source tools: **Wazuh** (SIEM), **Suricata** (IDS/IPS), and **Bettercap** (attack simulation).

---

## 🧪 Lab Overview

- **Attacker VM**: Kali Linux  to perform LAN-based attacks.
- **Victim VM**: Ubuntu with Suricata (IDS/packet capture), Zeek (network traffic analysis), and Wazuh agent..
- **Server VM**: Wazuh Manager for collecting and analyzing security events.

---

## 🎯 Objectives

- Simulate 11 Network-based attacks.
- Detect and analyze these attacks using Suricata and Wazuh.
- Collect `.pcap` files for packet-level investigation.
- Generate centralized logs and alerts.
- Document findings with step-by-step attack execution and detection.

---

## 🛠️ Tools Used

| Tool        | Role                                       |
|-------------|--------------------------------------------|
| Wazuh       | SIEM/log analysis (alert correlation)      |
| Suricata    | Network-based IDS/IPS + packet capture     |
| Zeek        | Network traffic analysis (detects advanced attacks missed by Suricata/Wazuh) |
| Bettercap   | Attack simulation (ARP/DNS spoofing, MITM) |
| hping3      | TCP SYN flood (DoS)                        |
|Hydra	      |Brute-force attack tool (used to automate login attempts on services like FTP, SSH, HTTP, etc.)|
| Yersinia    | Layer 2 attack simulation (e.g., DHCP, STP, CDP attacks) |
| Wireshark   | (Optional) PCAP analysis                   |
---



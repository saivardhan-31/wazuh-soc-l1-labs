# wazuh-soc-l1-labs
# Wazuh SOC L1 Home Lab — Security Monitoring & Detection Engineering

A practical SOC L1 home lab built using **Wazuh, Kali Linux, Ubuntu Server, Suricata, and Nmap** to practice security monitoring, detection, investigation, and incident response.

## Project Objective

Build a realistic SOC L1 environment to practice:

- Security alert monitoring
- Linux log analysis
- Attack detection
- Alert triage
- Event correlation
- MITRE ATT&CK mapping
- Incident investigation
- Containment and recovery

## Lab Architecture

```text
                ┌──────────────────┐
                │    Kali Linux    │
                │  Attacker /      │
                │  Wazuh Agent     │
                └────────┬─────────┘
                         │
                  Host-Only Network
                         │
                ┌────────▼─────────┐
                │  Ubuntu Server   │
                │                  │
                │  Wazuh Server    │
                │  Wazuh Indexer   │
                │  Wazuh Dashboard │
                │  Suricata        │
                └──────────────────┘
Labs Completed
Lab	Detection / Investigation
01	SSH Failed Login Detection
02	SSH Brute-Force Detection
03	SSH Login & Privileged Activity
04	Listening Port Detection
05	File Integrity Monitoring
06	Persistence Detection
07	User Account & Privilege Changes
08	Web Attack / SQL Injection Detection
09	Network Scanning using Suricata & Wazuh
10	Process & Command Execution Investigation
11	Multi-Event Attack Correlation
12	Full SOC L1 Incident Investigation & Response

## Key Technologies

SIEM: Wazuh
IDS: Suricata
Attacker / Testing: Kali Linux, Nmap
Server: Ubuntu Server
Monitoring: Wazuh Dashboard
Framework: MITRE ATT&CK
Virtualization: Oracle VirtualBox

## SOC Skills Demonstrated

Log collection and analysis
SSH attack detection
Brute-force investigation
Privilege escalation monitoring
File integrity monitoring
Persistence detection
Account and privilege monitoring
Web attack detection
Network scanning detection
Process and command investigation
Multi-event correlation
Incident timeline reconstruction
Incident containment and recovery

## Project Structure
Wazuh-SOC-L1-Home-Lab/
│
├── Lab-1-SSH-Failed-Login/
├── Lab-2-SSH-Brute-Force/
├── Lab-3-SSH-Privileged-Activity/
├── Lab-4-Listening-Port-Detection/
├── Lab-5-File-Integrity-Monitoring/
├── Lab-6-Persistence-Detection/
├── Lab-7-User-Account-Changes/
├── Lab-8-Web-Attack-Detection/
├── Lab-9-Network-Scanning-Detection/
├── Lab-10-Process-Command-Execution/
├── Lab-11-Multi-Event-Attack-Correlation/
└── Lab-12-Full-SOC-Incident-Investigation/

## Outcome

This project demonstrates hands-on experience with Wazuh SIEM monitoring, Linux security telemetry, detection engineering, Suricata IDS integration, alert triage, event correlation, MITRE ATT&CK, and SOC L1 incident response.

Each lab contains its own documentation, investigation evidence, and screenshots.

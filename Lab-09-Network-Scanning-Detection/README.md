# Lab 9 — Network Scanning Detection using Suricata and Wazuh

## Objective

To simulate network reconnaissance using Nmap from a Kali Linux system and detect the scanning activity using Suricata IDS integrated with Wazuh.

This lab demonstrates a SOC L1 workflow for detecting network scanning activity, analyzing IDS alerts, identifying the source and destination systems, and investigating the resulting Wazuh alerts.

---

## Lab Environment

| Component | Details |
|---|---|
| Attacker | Kali Linux |
| Target | Wazuh Server |
| SIEM | Wazuh |
| IDS | Suricata 7.0.3 |
| Scanning Tool | Nmap 7.98 |
| Virtualization | Oracle VirtualBox |
| Network | VirtualBox Host-Only Network |
| Kali IP | 192.168.158.3 |
| Wazuh Server IP | 192.168.158.5 |
| Suricata Log | `/var/log/suricata/eve.json` |

---

## Lab Architecture

```text
                    VirtualBox Host-Only Network
                         192.168.158.0/24

┌─────────────────────┐
│     Kali Linux      │
│                     │
│   192.168.158.3     │
│                     │
│       Nmap          │
│     SYN Scan        │
└──────────┬──────────┘
           │
           │ Network Scan
           ▼
┌──────────────────────────────────┐
│          Wazuh Server            │
│          192.168.158.5           │
│                                  │
│          Suricata IDS            │
│               │                  │
│               ▼                  │
│          eve.json                │
│               │                  │
│               ▼                  │
│             Wazuh                │
│               │                  │
│               ▼                  │
│        Security Alerts           │
└──────────────────────────────────┘

## Scenario

Network scanning is a common reconnaissance activity used to identify open ports and available services on a target system.

In this controlled SOC home lab, Kali Linux was used to perform an Nmap SYN scan against the Wazuh Server.

Suricata monitored the network traffic and detected traffic matching network scanning signatures. The detected events were written to eve.json and collected by Wazuh for security monitoring and investigation.

## Step 1 — Identify Network Configuration

The network interfaces on Kali Linux were checked using:

ip -br addr

Kali Linux Host-Only IP:

192.168.158.3

Wazuh Server Host-Only IP:

192.168.158.5

The Host-Only network was used for communication between Kali Linux and the Wazuh Server.

## Step 2 — Configure Suricata

For this lab, Suricata was configured with the Wazuh Server as the protected host.

HOME_NET: "[192.168.158.5]"
EXTERNAL_NET: "!$HOME_NET"

This configuration allowed the traffic flow to be treated as:

Kali Linux                         Wazuh Server
192.168.158.3  ────────────────►  192.168.158.5
   External                         HOME_NET

The Suricata configuration was tested using:

sudo suricata -T -c /etc/suricata/suricata.yaml

The configuration loaded successfully:

Configuration provided was successfully loaded. Exiting.
## Step 3 — Restart and Verify Suricata

Suricata was restarted to apply the configuration:

sudo systemctl restart suricata

The service was then verified:

sudo systemctl status suricata --no-pager

Result:

Active: active (running)

This confirmed that Suricata was running successfully.

## Step 4 — Perform Nmap SYN Scan

From Kali Linux, an Nmap SYN scan was performed against the Wazuh Server:

sudo nmap -sS -Pn -p 1-10000 --max-retries 1 192.168.158.5

The scan identified the following open ports:

22/tcp    open  ssh
443/tcp   open  https
1514/tcp  open
1515/tcp  open
9200/tcp  open
9300/tcp  open

The Nmap scan completed successfully.

Evidence

## Step 5 — Verify Suricata Alerts

Suricata stores its structured events in:

/var/log/suricata/eve.json

The event types were checked using:

sudo jq -r '.event_type' /var/log/suricata/eve.json | sort | uniq -c

The result included:

5 alert
24 dhcp
12160 flow
335 stats
41 tls

The important result was:

5 alert

This confirmed that Suricata generated five alert events during the Nmap scanning activity.

Evidence

## Step 6 — Analyze Suricata Detection Signatures

The generated Suricata alerts included the following ET SCAN signatures:

ET SCAN Suspicious inbound to mySQL port 3306
ET SCAN Suspicious inbound to MSSQL port 1433
ET SCAN Potential VNC Scan 5800-5820
ET SCAN Suspicious inbound to PostgreSQL port 5432
ET SCAN Suspicious inbound to Oracle SQL port 1521

The observed traffic included:

Source IP:      192.168.158.3
Destination IP: 192.168.158.5

The traffic therefore originated from Kali Linux and targeted the Wazuh Server.

These alerts indicate detected scanning or probing activity. They do not by themselves indicate that the corresponding services were compromised.

## Step 7 — Verify Alerts in Wazuh

The Suricata alerts were successfully forwarded to Wazuh.

The Wazuh Dashboard displayed the Suricata alerts generated during the Nmap scan.

The observed Wazuh alerts included:

Rule ID	Level	Detection
86601	3	ET SCAN Suspicious inbound to Oracle SQL port 1521
86601	3	ET SCAN Suspicious inbound to MSSQL port 1433
86601	3	ET SCAN Suspicious inbound to PostgreSQL port 5432
86601	3	ET SCAN Potential VNC Scan 5800-5820
86601	3	ET SCAN Suspicious inbound to mySQL port 3306

The Wazuh Dashboard showed these events under the wazuh-server agent.

Evidence

## Alert Analysis
Field	Value
Source IP	192.168.158.3
Source System	Kali Linux
Destination IP	192.168.158.5
Destination System	Wazuh Server
Network Interface	enp0s8
Detection Source	Suricata
Event Type	alert
Wazuh Rule	86601
Wazuh Level	3
Log Source	/var/log/suricata/eve.json
Activity	Network scanning

The Nmap SYN scan generated network traffic that matched Suricata's network scanning detection signatures.

Wazuh received the Suricata events and generated corresponding alerts using Rule 86601.

## SOC L1 Investigation

A SOC L1 analyst can investigate the alert using the following process.

## 1. Identify the Source
192.168.158.3

The source was the Kali Linux system that generated the test traffic.

## 2. Identify the Target
192.168.158.5

The target was the Wazuh Server.

3. Identify the Activity

The activity was an Nmap TCP SYN scan against ports 1–10000.

## 4. Review Detection Evidence

The analyst reviews:

Source IP
Destination IP
Destination ports
Suricata signatures
Event timestamp
Network interface
Wazuh alert details

## 5. Correlate the Events
Nmap SYN Scan
      ↓
Network Traffic
      ↓
Suricata Detection
      ↓
eve.json
      ↓
Wazuh Rule 86601
      ↓
Wazuh Dashboard Alert
## 6. Determine the Context

The scanning activity was intentionally generated as part of an authorized SOC home lab.

Therefore, the activity was expected within this controlled environment.

## MITRE ATT&CK Mapping
T1046 — Network Service Scanning

The simulated activity maps to:

T1046 — Network Service Scanning

The activity involved scanning network ports to identify available services on the target system.

Nmap SYN Scan
      ↓
Port Scanning
      ↓
Network Service Scanning
      ↓
Suricata Detection
      ↓
Wazuh Alert
      ↓
SOC Investigation

##Classification

Category	Result
Activity	Network Scanning / Reconnaissance
Source	Kali Linux
Source IP	192.168.158.3
Target	Wazuh Server
Target IP	192.168.158.5
Detection	Suricata IDS
SIEM	Wazuh
Wazuh Rule	86601
Wazuh Level	3
MITRE ATT&CK	T1046
Environment	Authorized SOC Home Lab

##Skills Demonstrated

Nmap
TCP SYN scanning
Network reconnaissance detection
Suricata IDS
Wazuh SIEM
EVE JSON analysis
Network traffic analysis
IDS signature analysis
Source and destination IP analysis
Port analysis
Security alert investigation
SOC L1 alert triage
MITRE ATT&CK mapping
Detection engineering

## Detection Workflow
Kali Linux
     |
     | Nmap SYN Scan
     v
Network Traffic
     |
     v
Suricata IDS
     |
     | ET SCAN Detection
     v
eve.json
     |
     v
Wazuh Rule 86601
     |
     v
Wazuh Dashboard
     |
     v
SOC L1 Investigation
     |
     v
MITRE ATT&CK T1046

##Key Takeaways

Nmap SYN scans generate network traffic that can be detected by an IDS.
Suricata can identify network scanning activity using detection signatures.
Suricata records structured security events in EVE JSON format.
Wazuh can collect Suricata alerts and display them in the SIEM dashboard.
Source and destination IP addresses are important during SOC alert triage.
IDS signatures can identify suspicious traffic toward commonly associated service ports.
A network scanning alert does not by itself indicate successful compromise.
SOC analysts should investigate alerts in the context of the environment.
Authorized security testing should be distinguished from malicious activity.

## Conclusion

This lab successfully demonstrated network scanning detection using Nmap, Suricata, and Wazuh.

A controlled Nmap SYN scan was launched from Kali Linux against the Wazuh Server. Suricata detected the scanning activity and generated five alert events.

The Suricata alerts were recorded in eve.json and successfully forwarded to Wazuh. Wazuh displayed the resulting alerts using Rule 86601, including multiple ET SCAN signatures associated with the scanning activity.

## The complete SOC detection workflow was demonstrated:

Nmap Network Scan
        ↓
Suricata IDS Detection
        ↓
Suricata EVE JSON
        ↓
Wazuh Rule 86601
        ↓
Wazuh Security Alert
        ↓
SOC L1 Investigation
        ↓
MITRE ATT&CK T1046

This lab demonstrates practical SOC L1 skills in network monitoring, IDS alert analysis, SIEM investigation, detection engineering, and MITRE ATT&CK mapping.

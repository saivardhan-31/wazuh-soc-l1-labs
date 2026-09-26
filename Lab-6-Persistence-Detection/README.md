# Lab 6 — Persistence Detection using Wazuh

## Objective

The objective of this lab was to simulate a controlled Linux persistence technique using a cron configuration file and verify that Wazuh File Integrity Monitoring (FIM) detects the change in real time.

This lab helped me practice:

- Linux persistence concepts
- Cron monitoring
- Wazuh FIM
- SOC L1 alert investigation
- MITRE ATT&CK awareness
- Alert classification and cleanup

---

## Lab Environment

| Component | Details |
|---|---|
| SIEM | Wazuh |
| Wazuh Server | Ubuntu Server 24.04.5 LTS |
| Server Resources | 4 CPU, 8 GB RAM |
| Endpoint | Kali Linux 2025.4 |
| Architecture | x86_64 |
| Wazuh Agent | `kali-soc-lab` |
| Wazuh Manager | `wazuh-server` |
| Monitoring | Wazuh FIM / Syscheck |
| Monitored Directory | `/etc/cron.d/` |
| Mode | Real-time |

---

## Lab Architecture
```text
Kali Linux
    |
    | Cron persistence configuration
    ↓
/etc/cron.d/
    |
    | File change
    ↓
Wazuh FIM
    |
    ↓
Wazuh Agent
    |
    ↓
Wazuh Manager
    |
    ↓
Wazuh Dashboard
    |
    ↓
SOC L1 Investigation

## Scenario

A controlled cron persistence configuration was created on my own Kali Linux machine.

The test file was:

/etc/cron.d/lab6-persistence-test

The cron entry was intentionally harmless and executed:

/usr/bin/touch /tmp/lab6-persistence-test

The existing cron files were not modified.

The test was performed only against my own lab environment.

## Step 1 — Check Existing Cron Configuration

I first reviewed the existing cron directory:

sudo ls -lah /etc/cron.d/

The existing configuration was left untouched.

## Step 2 — Create Controlled Persistence

I created a dedicated cron configuration file:

echo '* * * * * root /usr/bin/touch /tmp/lab6-persistence-test' | sudo tee /etc/cron.d/lab6-persistence-test

Then I set the required permissions:

sudo chmod 644 /etc/cron.d/lab6-persistence-test

The file was verified using:

sudo cat /etc/cron.d/lab6-persistence-test
## Step 3 — Wazuh Persistence Detection

Wazuh Dashboard → Threat Hunting → Events

Wazuh FIM detected the new cron configuration in real time.

Alert Details
Agent: kali-soc-lab
Decoder: syscheck_new_entry
File: /etc/cron.d/lab6-persistence-test
Description: File added to the system
Rule ID: 554
Rule Level: 5
Mode: Real-time
Location: syscheck

This confirmed that Wazuh detected the new persistence configuration.

## Step 4 — SOC L1 Investigation

A new file under /etc/cron.d/ can be relevant to a Linux persistence investigation.

I reviewed:

File path
File contents
Detection source
Rule ID and severity
Whether the change was authorized
Classification
Benign / Authorized Security Testing

The cron entry was intentionally created as part of this lab.

## Step 5 — Remove Persistence

After validating the detection, I removed only the test file:

sudo rm /etc/cron.d/lab6-persistence-test

Wazuh FIM subsequently detected the file deletion.

Deletion Alert
Decoder: syscheck_deleted
Description: File deleted
Rule ID: 553
Rule Level: 7
Mode: Real-time

The Wazuh event also displayed MITRE mappings T1070.004 and T1485 from the deletion rule.

These mappings belong to Wazuh's deletion rule. The deletion itself was an authorized cleanup action.

## MITRE ATT&CK

The lab demonstrates Linux persistence through scheduled-task/cron configuration.

The deletion event displayed:

T1070.004
T1485

The primary focus of this lab was FIM-based detection of a persistence configuration change.

## SOC L1 Skills Practiced
Linux persistence monitoring
Cron investigation
Wazuh FIM
Real-time file monitoring
Alert triage
Rule ID and severity analysis
MITRE ATT&CK awareness
Event classification
Security evidence documentation
## Investigation Workflow
Persistence Created
        ↓
Wazuh FIM Detection
        ↓
Alert Triage
        ↓
File Investigation
        ↓
Classification
        ↓
Persistence Removed
        ↓
FIM Cleanup Detection
## Key Takeaways
Cron configuration can be relevant during Linux persistence investigations.
Wazuh FIM can detect file changes in real time.
Rule 554 detected the new persistence file.
Rule 553 detected its removal.
File creation alone does not prove malicious activity.
SOC analysts must validate whether the change is authorized.
## Conclusion

This lab demonstrated how Wazuh FIM can detect changes to Linux cron configuration that may be relevant to persistence investigations.

I created a controlled persistence configuration, investigated the Wazuh alert from a SOC L1 perspective, classified the event, and removed the test configuration.

## Skills demonstrated: Linux Persistence → Wazuh FIM → Alert Triage → Investigation → Classification → Cleanup

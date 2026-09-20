# Lab 7 — User Account & Privilege Changes using Wazuh

## Objective

The objective of this lab was to simulate controlled Linux user account creation and privilege changes and investigate the resulting Wazuh alerts.

This lab helped me practice:

- Linux user account monitoring
- Privilege and group changes
- Wazuh alert investigation
- FIM monitoring
- MITRE ATT&CK awareness
- SOC L1 alert classification and cleanup

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
| Log Source | systemd-journald |
| Monitoring | Wazuh FIM / Syscheck |

---

## Lab Architecture
Kali Linux
    |
    | User account activity
    ↓
systemd-journald
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

A controlled test user named lab7user was created on my own Kali Linux machine.

The account was then added to the sudo group to simulate a privilege change.

The existing users sai and support-agent were not modified.

The test was performed only against my own lab environment.


## Step 1 — Check Existing Sudo Users

I first checked the existing sudo group:

getent group sudo

Existing members:

sai
support-agent

These accounts were left unchanged.

## Step 2 — Create Test User

I created a dedicated test account:

sudo useradd -m lab7user

The account was verified:

id lab7user

The user was created with:

UID: 1005
GID: 1005

## Step 3 — Add Sudo Privilege

The test account was added to the sudo group:

sudo usermod -aG sudo lab7user

The privilege change was verified:

groups lab7user

Output:

lab7user : lab7user sudo

## Step 4 — Wazuh Account Detection

Wazuh Dashboard → Threat Hunting → Events

Wazuh detected the account creation.

Rule 5902
Description: New user added to the system
Username: lab7user
UID: 1005
GID: 1005
Decoder: useradd
Rule ID: 5902
Rule Level: 8
MITRE ATT&CK: T1136
Technique: Create Account
Tactic: Persistence

Wazuh also detected the new group created for the account.

Rule 5901
Description: New group added to the system
Group: lab7user
GID: 1005
Rule ID: 5901
Rule Level: 8
Decoder: useradd

## Step 5 — FIM Supporting Evidence

Wazuh also generated multiple:

Integrity checksum changed

alerts with:

Rule ID: 550
Rule Level: 7

These alerts were generated because the user and group operations modified monitored Linux account-related files.

These were treated as supporting evidence for the account-management activity.

## SOC L1 Investigation

A newly created user should be investigated because unauthorized accounts can provide access or persistence.

I reviewed:

Username
UID/GID
Account creation event
Group membership
Sudo membership
Wazuh rule ID and severity
Related FIM events
Classification
Benign / Authorized Security Testing

The account was intentionally created as part of this lab.

## Step 6 — Cleanup

After completing the investigation, I removed the test account:

sudo userdel -r lab7user

The account was verified as removed:

id lab7user

Result:

id: ‘lab7user’: no such user

The existing user accounts were not modified.

## MITRE ATT&CK

Wazuh mapped the account creation alert to:

T1136 — Create Account

The alert was associated with the Persistence tactic.

The primary focus of this lab was monitoring and investigating unauthorized or unexpected account creation and privilege changes.

## SOC L1 Skills Practiced

Linux account monitoring
User creation detection
Group membership analysis
Sudo privilege monitoring
Wazuh alert triage
FIM analysis
MITRE ATT&CK mapping
Event classification
Security evidence documentation
Cleanup and verification

## Investigation Workflow
User Created
     ↓
Wazuh Detection
     ↓
Account Investigation
     ↓
Privilege Verification
     ↓
Alert Classification
     ↓
Test Account Removed
     ↓
Cleanup Verification

## Key Takeaways
Unexpected user creation can be an important SOC alert.
Wazuh can detect Linux account creation through authentication/system logs.
Rule 5902 detected the new user with Level 8 severity.
Rule 5901 detected the new group.
Group membership should be checked when investigating new accounts.
FIM alerts can provide supporting evidence of account-related system changes.
Account creation alone does not prove malicious activity; analyst validation is required.

## Conclusion

This lab demonstrated how Wazuh can detect and investigate Linux user account creation and related privilege changes.

I created a controlled test account, added it to the sudo group, investigated the resulting Wazuh alerts, verified the privilege change, and removed the test account.

Skills demonstrated: User Monitoring → Privilege Analysis → Wazuh Detection → SOC L1 Investigation → Classification → Cleanup

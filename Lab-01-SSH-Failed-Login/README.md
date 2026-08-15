# Lab 1 — SSH Failed Login Detection using Wazuh

## Objective

The objective of this lab was to detect and investigate a failed SSH authentication attempt on a Linux endpoint using Wazuh.

This lab helped me practice:

- Linux authentication log analysis
- SSH monitoring
- Wazuh alert investigation
- Rule and severity analysis
- Source IP and username identification
- MITRE ATT&CK mapping
- Basic SOC L1 alert triage

---

## Lab Environment

| Component | Details |
|---|---|
| SIEM | Wazuh Cloud |
| Endpoint | Kali Linux 2025.4 |
| Architecture | x86_64 |
| Wazuh Agent | `kali-soc-lab` |
| Log Collection | systemd-journald |
| Monitored Service | SSH |

---

## Lab Architecture

```text
Kali Linux
    |
    | SSH authentication event
    ↓
systemd-journald
    |
    | Log collection
    ↓
Wazuh Agent
    |
    ↓
Wazuh Cloud
    |
    ↓
Wazuh Dashboard
    |
    ↓
SOC L1 Investigation
```


Scenario

A controlled failed SSH authentication attempt was generated against my own Kali Linux machine.

The purpose was to verify that:

Kali records the SSH authentication event.
The Wazuh Agent collects the event.
Wazuh Cloud receives and analyzes the event.
Wazuh generates a security detection.
The event can be investigated from a SOC L1 perspective.

#The test was performed only against my own lab environment.

Step 1 — Generate Failed SSH Login

I used the following command:

ssh wronguser@127.0.0.1

Here:

wronguser was an intentionally nonexistent username.
127.0.0.1 refers to my own Kali Linux machine.

I entered an incorrect password, resulting in a failed SSH authentication attempt.

Step 2 — Verify the Linux Log

Kali Linux was using systemd-journald for authentication logging.

I checked the SSH logs using:

sudo journalctl -u ssh --since "2 minutes ago" --no-pager

The log showed a failed authentication event similar to:

Failed password for invalid user wronguser from 127.0.0.1
Evidence

Step 3 — Investigate the Wazuh Event

The event was found in:

Wazuh Dashboard → Threat Hunting → Events

The event was associated with my Kali endpoint:

Agent: kali-soc-lab
Wazuh Alert Details
Field	Result
Agent	kali-soc-lab
Rule ID	5710
Rule Level	5
Rule Description	sshd: Attempt to login using a non-existent user
Source IP	127.0.0.1
Source User	wronguser
Log Source	journald
Decoder	sshd
Wazuh Events

Detailed Wazuh Alert

Step 4 — Alert Analysis

Wazuh generated Rule 5710 with Level 5 severity.

The rule description was:

sshd: Attempt to login using a non-existent user

The event showed that the SSH service received a login attempt using the nonexistent username:

wronguser

The source IP was:

127.0.0.1

Since 127.0.0.1 is the loopback address, this activity originated from my own Kali machine during the controlled test.

MITRE ATT&CK Mapping

Wazuh mapped this activity to:

MITRE Information	Result
Tactic	Credential Access
Technique	Password Guessing
Technique ID	T1110.001
SSH Technique	T1021.004

This indicates that the detected activity is related to credential guessing involving SSH.

SOC L1 Investigation
What happened?

A failed SSH authentication attempt was made using a nonexistent username.

Which user was targeted?
wronguser
What was the source IP?
127.0.0.1
Which Wazuh rule detected it?
Rule ID: 5710
What was the alert level?
Level: 5
Is this a brute-force attack?

No.

A single failed SSH authentication attempt is not enough evidence to classify the activity as brute force.

For a brute-force investigation, I would look for:

Multiple failed login attempts
Repeated attempts within a short period
Same source IP
Multiple targeted usernames
Successful login after repeated failures
Suspicious activity after authentication
Alert Classification

Classification: Benign / Controlled Test

The event was intentionally generated against my own Kali Linux SOC laboratory environment.

There was no unauthorized access.

The purpose of the event was to verify the complete Wazuh log collection and detection pipeline.

#SOC L1 Skills Practiced

Through this lab, I practiced:

Linux authentication log analysis
SSH log monitoring
systemd-journald investigation
Wazuh SIEM investigation
Alert triage
Rule ID analysis
Severity analysis
Source IP identification
Username identification
MITRE ATT&CK mapping
Differentiating a single failed login from brute-force activity
Basic incident classification

#Investigation Workflow

Failed SSH Login
       ↓
Kali Linux
       ↓
systemd-journald
       ↓
Wazuh Agent
       ↓
Wazuh Cloud
       ↓
Wazuh Alert
       ↓
SOC L1 Investigation
       ↓
Benign Controlled Test

#Key Takeaways

SSH authentication logs are useful for detecting suspicious login activity.
Kali Linux uses systemd-journald for this lab's authentication logging.
Wazuh successfully collected and analyzed the SSH event.
Wazuh Rule 5710 detected the nonexistent-user SSH login attempt.
The alert was assigned Level 5.
A single failed login should not automatically be classified as brute force.
SOC analysts need to correlate multiple events and investigate the surrounding activity before determining whether an incident is malicious.

#Conclusion

This lab successfully demonstrated the complete process of generating, collecting, detecting, and investigating a failed SSH authentication event.

The event was successfully collected from Kali Linux by the Wazuh Agent and displayed in Wazuh Cloud.

The activity was classified as a benign controlled test because it was intentionally generated within my own SOC practice environment.

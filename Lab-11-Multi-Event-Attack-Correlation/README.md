# Lab 11 — Multi-Event Attack Correlation

## Objective

To investigate and correlate multiple security events generated during a controlled SSH attack simulation using Wazuh.

The lab demonstrates how a SOC L1 analyst can correlate:

- Multiple failed SSH authentication attempts
- SSH brute-force detection
- Successful SSH authentication
- Privileged command execution

into a single security incident timeline.

---

## Lab Environment

| Component | Details |
|---|---|
| Attacker / Test System | Kali Linux |
| Wazuh Agent | `kali-soc-lab` |
| Wazuh Server | Ubuntu Server |
| SIEM | Wazuh |
| Log Source | Linux `journald` |
| Kali IP | `192.168.158.3` |
| Wazuh Server IP | `192.168.158.5` |
| Test Source IP | `127.0.0.1` |

---

## Lab Architecture

```text
                    Kali Linux
                 192.168.158.3
                       |
                       | Wazuh Agent
                       |
                       v
                +---------------+
                | Wazuh Server  |
                | 192.168.158.5 |
                +-------+-------+
                        |
                        v
                Wazuh Dashboard
                        |
                        v
             Multi-Event Correlation

## Scenario

A controlled SSH attack sequence was simulated on the Kali Linux system.

The activity consisted of:

Multiple Failed SSH Attempts
            |
            v
SSH Brute-Force Detection
            |
            v
Successful SSH Authentication
            |
            v
Privileged Command Execution

The objective was to determine whether these events could be correlated into a single security incident instead of being investigated independently.

## Step 1 — Generate Failed SSH Attempts

The SSH service was started on Kali Linux.

Multiple SSH authentication attempts were then performed using a non-existent user.

Wazuh generated multiple authentication-related alerts.

Observed Alerts

Rule 5710 — Level 5

sshd: Attempt to login using a non-existent user

Rule 5503 — Level 5

PAM: User login failed.

Rule 2502 — Level 10

syslog: User missed the password more than one time

Most importantly, Wazuh generated:

Rule 5712 — Level 10

sshd: brute force trying to get access to the system. Non existent user.

This established the initial suspicious SSH authentication activity.

Evidence

## Step 2 — Successful SSH Authentication

After the failed authentication activity, a controlled successful SSH login was performed using the legitimate Kali user.

Wazuh recorded:

Rule ID: 5715
Description: sshd: authentication success.
User: sai
Source IP: 127.0.0.1
Decoder: sshd
Location: journald

The underlying event contained:

Accepted password for sai from 127.0.0.1

The Wazuh event was mapped to:

T1078 — Valid Accounts
T1021 — Remote Services

The successful authentication became important because it occurred after the earlier failed authentication and brute-force activity.

Evidence

## Step 3 — Privileged Command Execution

After the successful SSH login, a controlled privileged command was executed:

sudo touch /tmp/lab11-correlation-test

Wazuh detected the activity using:

Rule 5402 — Level 3

Successful sudo to ROOT executed.

The event contained:

Command: /usr/bin/touch /tmp/lab11-correlation-test
Source User: sai
Destination User: root
Working Directory: /home/sai
TTY: pts/1
Decoder: sudo
Location: journald

## MITRE ATT&CK mapping:

T1548.003 — Sudo and Sudo Caching

Tactics:

Privilege Escalation
Defense Evasion
Evidence

## Alert Analysis

The events were analyzed as a sequence rather than as isolated alerts.

Time	Event	Rule	Level
~09:20	Failed SSH authentication	5710	5
~09:20	SSH brute-force detection	5712	10
09:40	Successful SSH authentication	5715	3
09:46	Privileged command execution	5402	3

The important investigative finding was the relationship between these events.

Failed SSH Authentication
          |
          v
SSH Brute-Force Detection
          |
          v
Successful SSH Authentication
          |
          v
Privileged Command Execution

This sequence provides more context than investigating each alert individually.

## SOC L1 Investigation
1. Investigate Authentication Failures

The investigation began with repeated failed SSH authentication attempts.

Wazuh generated Rule 5710 and subsequently detected brute-force behavior using Rule 5712.

Rule: 5712
Level: 10
Description: SSH brute force trying to access the system
2. Investigate Successful Authentication

A successful SSH authentication was identified after the failed authentication activity.

Rule: 5715
User: sai
Source IP: 127.0.0.1

The successful authentication required additional investigation because it followed the previous failed authentication attempts.

3. Investigate Post-Authentication Activity

The next relevant event was privileged command execution.

Source User: sai
Destination User: root
Command: /usr/bin/touch /tmp/lab11-correlation-test

Wazuh mapped this activity to:

T1548.003 — Sudo and Sudo Caching
4. Correlate the Events

The complete event sequence was reconstructed as:

[Failed SSH Attempts]
          |
          v
[SSH Brute-Force Detection]
          |
          v
[Successful SSH Authentication]
          |
          v
[Privileged Command Execution]
          |
          v
[Correlated Security Incident]

## The correlation was based on:

Event timestamps
User identity
Source IP
Authentication activity
Post-authentication activity
Privileged command execution
Evidence

## MITRE ATT&CK Mapping
Activity	MITRE ATT&CK Technique
Successful SSH authentication	T1078 — Valid Accounts
SSH remote access	T1021 — Remote Services
Privileged sudo execution	T1548.003 — Sudo and Sudo Caching
Incident Classification

## Incident Type:

SSH Authentication Attack / Suspicious Privileged Activity

Initial Detection:

SSH brute-force activity

Post-Authentication Activity:

Successful SSH login followed by privileged command execution

Environment:

Controlled SOC laboratory simulation

This was a controlled localhost simulation performed for security monitoring and detection engineering. It does not represent an actual external compromise.

## SOC L1 Investigation Workflow
Alert Detection
      |
      v
Review Authentication Failures
      |
      v
Identify Brute-Force Activity
      |
      v
Check Successful Authentication
      |
      v
Identify User and Source IP
      |
      v
Review Post-Login Activity
      |
      v
Check Privileged Commands
      |
      v
Correlate Timeline
      |
      v
Map to MITRE ATT&CK
      |
      v
Classify Incident
      |
      v
Document Evidence
Evidence Summary

## The investigation confirmed:

Multiple failed SSH authentication attempts.
Wazuh generated a Level 10 SSH brute-force alert.
A successful SSH authentication occurred afterward.
The successful authentication originated from 127.0.0.1.
The authenticated user was sai.
A privileged sudo command was subsequently executed.
Wazuh recorded the command and source/destination users.
MITRE ATT&CK mappings were available for the observed activities.
The temporary test file was removed after the investigation.

## Relevant Wazuh Rules
Rule ID	Level	Description	Purpose
5710	5	SSH login using non-existent user	Failed authentication
5712	10	SSH brute-force detection	Brute-force activity
5715	3	SSH authentication success	Successful authentication
5402	3	Successful sudo to ROOT executed	Privileged activity

Rule 533 was observed during the lab because starting the SSH service changed the listening-port state. It was not included in the attack correlation because it was unrelated to the SSH authentication sequence.

## Skills Demonstrated

Multi-event security alert correlation
SSH authentication monitoring
Brute-force detection
Linux log analysis
Wazuh SIEM investigation
Authentication investigation
Privileged activity analysis
Timeline reconstruction
MITRE ATT&CK mapping
SOC L1 incident classification
Security evidence documentation
Alert relevance analysis

## Key Takeaways
A single security alert may not provide enough context to understand an incident.
Multiple authentication failures can provide important context when followed by successful authentication.
Successful authentication should be investigated together with the surrounding events.
Post-authentication privileged activity should be reviewed.
Wazuh provides the event data required to reconstruct a security timeline.
SOC analysts should correlate events using timestamps, users, source IPs and activity.
Not every alert generated during the same period is necessarily part of the same incident.
Rule 533 was excluded from the correlation because it represented a listening-port change caused by starting SSH.

## Conclusion

This lab demonstrated a complete multi-event SOC investigation using Wazuh.

Instead of analyzing individual alerts separately, the events were correlated into a single timeline:

Failed SSH Attempts
        |
        v
SSH Brute-Force Detection
        |
        v
Successful SSH Authentication
        |
        v
Privileged Command Execution
        |
        v
SOC L1 Correlation and Investigation

The lab demonstrates how a SOC L1 analyst can move from individual security alerts to a correlated incident timeline, investigate post-authentication activity, map observed behavior to MITRE ATT&CK, and document the investigation.

# Lab 2 — SSH Brute-Force Detection and Investigation using Wazuh

## Objective

The objective of this lab was to simulate repeated SSH authentication failures and investigate how Wazuh detects SSH brute-force activity.

This lab helped me practice:

- Detecting repeated SSH authentication failures
- Investigating Wazuh correlation rules
- Analyzing Linux SSH logs
- Identifying source IP and targeted username
- Understanding alert severity
- Checking for successful authentication after an attack
- Mapping activity to MITRE ATT&CK
- Performing SOC L1 alert investigation and classification

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
Kali Linux
    |
    | Multiple SSH authentication attempts
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
Wazuh Detection & Correlation
    |
    ↓
SOC L1 Investigation

## Scenario

A controlled SSH brute-force simulation was performed against my own Kali Linux laboratory machine.

The purpose was to generate multiple failed SSH authentication attempts and observe whether Wazuh could correlate the events and identify brute-force behavior.

The test was performed only against my own lab environment.

## Step 1 — Generate Multiple Failed SSH Attempts

I generated repeated SSH authentication attempts using the following command:

for i in {1..10}; do ssh -o BatchMode=yes -o StrictHostKeyChecking=no wronguser@127.0.0.1; done

## The test used:

Username: wronguser
Destination: 127.0.0.1
Number of attempts: 10

The username was intentionally nonexistent on the Kali Linux endpoint.

The repeated authentication attempts generated multiple SSH events.

## Step 2 — Verify the SSH Logs

Kali Linux was using systemd-journald for SSH logging.

I checked the SSH authentication events using:

sudo journalctl -u ssh --since "1 hour ago" --no-pager | grep -Ei "Accepted|Failed|Invalid user"

The logs showed multiple invalid-user SSH attempts.

Example:

Aug 16 17:48:44 vbox sshd-session[4575]: Invalid user wronguser from 127.0.0.1 port 34300
Aug 16 17:48:44 vbox sshd-session[4578]: Invalid user wronguser from 127.0.0.1 port 34306
Aug 16 17:48:44 vbox sshd-session[4581]: Invalid user wronguser from 127.0.0.1 port 34316

The attempts occurred within approximately one second, from the same source IP and against the same username.

The connections were closed during the pre-authentication stage.

Evidence

## Step 3 — Investigate Wazuh Events

The generated events were investigated in:

Wazuh Dashboard → Threat Hunting → Events

Multiple SSH authentication events were observed from the Kali endpoint.

The events included individual failed-login detections and a correlated brute-force detection.

Individual SSH Detection

Wazuh generated Rule 5710 for individual invalid-user SSH attempts:

Field	Result
Rule ID	5710
Rule Level	5
Description	sshd: Attempt to login using a non-existent user

These individual events were then correlated by Wazuh.

## Step 4 — Wazuh Brute-Force Detection

Wazuh generated a correlated brute-force alert.

Alert Details
Field	Result
Agent	kali-soc-lab
Source IP	127.0.0.1
Source User	wronguser
Decoder	sshd
Rule ID	5712
Rule Level	10
Rule Frequency	8
Rule Description	sshd: brute force trying to get access to the system. Non existent user.
MITRE ID	T1110
MITRE Tactic	Credential Access
MITRE Technique	Brute Force

The previous_output field showed multiple invalid-user SSH events from the same source:

Invalid user wronguser from 127.0.0.1

This demonstrated that Wazuh correlated the repeated authentication failures and generated a higher-severity brute-force detection.

Evidence — Wazuh Events

Evidence — Wazuh Rule 5712

## Step 5 — SOC L1 Investigation
1. What happened?

Multiple SSH authentication attempts were made using the nonexistent username wronguser.

2. What was the source IP?
127.0.0.1

This is the loopback address of the Kali Linux machine used for the controlled test.

3. Which username was targeted?
wronguser

The account does not exist on the endpoint.

4. How many attempts were generated?

I generated:

10 failed SSH attempts

The attempts occurred within approximately one second.

5. Did Wazuh detect brute-force behavior?

Yes.

Wazuh correlated the repeated SSH authentication failures and generated:

Rule ID: 5712
Level: 10

with the description:

sshd: brute force trying to get access to the system. Non existent user.
6. Did a successful login occur?

I checked the SSH logs for both failed and successful authentication events using:

sudo journalctl -u ssh --since "1 hour ago" --no-pager | grep -Ei "Accepted|Failed|Invalid user"

The output showed the repeated invalid-user attempts, but no:

Accepted password

or:

Accepted publickey

event was observed.

Therefore:

No successful SSH authentication was observed after the brute-force attempts.

7. Was there a confirmed account compromise?

No.

There was no evidence of a successful SSH login following the detected brute-force activity.

## MITRE ATT&CK Mapping

Wazuh mapped the brute-force activity to:

MITRE Information	Result
Tactic	Credential Access
Technique	Brute Force
Technique ID	T1110

The detection is associated with repeated attempts to gain access by guessing credentials.

## Alert Classification

Detection Classification: True Positive

Wazuh correctly identified the repeated SSH authentication attempts as brute-force behavior.

Activity Classification: Benign / Authorized Security Test

The activity was intentionally generated against my own Kali Linux SOC laboratory environment.

Therefore:

True Positive Detection
        +
Authorized / Controlled Activity
        =
Benign Security Test

There was no confirmed account compromise.

## SOC L1 Investigation Workflow
Multiple SSH Login Failures
          ↓
Kali systemd-journald
          ↓
Wazuh Agent
          ↓
Wazuh Cloud
          ↓
Individual Rule 5710 Alerts
          ↓
Event Correlation
          ↓
Rule 5712 / Level 10
          ↓
SSH Brute-Force Detection
          ↓
Check for Successful Login
          ↓
No Successful Login Observed
          ↓
No Confirmed Account Compromise
          ↓
Benign Controlled Test
SOC L1 Skills Practiced

## Through this lab, I practiced:

SSH brute-force detection
Linux authentication log analysis
systemd-journald investigation
Wazuh SIEM investigation
Alert correlation
Wazuh rule analysis
Severity analysis
Source IP identification
Username identification
MITRE ATT&CK mapping
Checking for successful authentication
Determining whether an account was compromised
True positive detection analysis
SOC L1 incident classification

## Key Takeaways
Multiple failed SSH authentication attempts can indicate brute-force activity.
Wazuh can correlate individual SSH authentication failures.
Rule 5710 identifies individual invalid-user SSH attempts.
Rule 5712 identifies correlated SSH brute-force behavior.
Rule 5712 generated a Level 10 alert in this investigation.
The source IP and targeted username are important investigation indicators.
A SOC analyst should always check whether a successful login occurred after repeated failures.
A true positive detection does not necessarily mean a real compromise occurred.
Context and correlation are essential when classifying security alerts.

## Conclusion
This lab demonstrated the complete detection and investigation process for SSH brute-force activity.

Multiple failed SSH authentication attempts were generated against the Kali Linux endpoint. Wazuh successfully collected the events and correlated the repeated authentication failures.

Wazuh generated Rule 5712 at Level 10, identifying the activity as SSH brute-force behavior and mapping it to MITRE ATT&CK T1110 — Brute Force.

A review of the Linux SSH logs showed repeated invalid-user attempts from 127.0.0.1, but no successful authentication was observed afterward.

The detection was therefore classified as a true positive detection of brute-force behavior, while the activity itself was classified as a benign authorized security test because it was intentionally generated in my SOC laboratory.

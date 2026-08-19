# Lab 3 — Successful SSH Login & Privileged Activity Investigation using Wazuh

## Objective

The objective of this lab was to investigate a successful SSH authentication followed by privileged activity on a Linux endpoint using Wazuh.

This lab helped me practice:

- Successful SSH authentication monitoring
- Linux authentication log analysis
- Wazuh alert investigation
- Correlation of authentication and post-login activity
- Privileged activity investigation
- Sudo activity analysis
- Rule and severity analysis
- MITRE ATT&CK mapping
- SOC L1 alert triage and classification

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
| Privileged Command | `sudo whoami` |

---

## Lab Architecture

Kali Linux
    |
    | Successful SSH authentication
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
Wazuh Detection
    |
    ↓
SOC L1 Correlation
    |
    ↓
Post-Login Privileged Activity

## Scenario

A controlled successful SSH login was performed against my own Kali Linux laboratory machine.

After successfully authenticating through SSH, a controlled privileged command was executed using sudo.

The purpose was to investigate:

Who successfully logged in?
Where did the login originate?
What happened after authentication?
Was privileged activity performed?
Was the activity suspicious or authorized?

The test was performed only against my own lab environment.

## Step 1 — Generate a Successful SSH Login

I established an SSH connection to my own Kali Linux machine using:

ssh sai@127.0.0.1

After providing the correct password, the SSH session was successfully established.

The successful authentication was recorded in the Linux SSH logs.

The relevant event was:

Accepted password for sai from 127.0.0.1 port 40092 ssh2

This confirmed:

Username: sai
Source IP: 127.0.0.1
Authentication: Successful
Service: SSH
## Step 2 — Generate Controlled Privileged Activity

After successfully logging in through SSH, I executed:

sudo whoami

The command returned:

root

This demonstrated that the authenticated user executed a command with root privileges through sudo.

The Linux log recorded:

sai : TTY=pts/1 ; PWD=/home/sai ; USER=root ; COMMAND=/usr/bin/whoami
## Step 3 — Investigate the Successful SSH Login in Wazuh

The successful SSH authentication event was investigated in:

Wazuh Dashboard → Threat Hunting → Events

Wazuh SSH Alert Details
Field	Result
Agent	kali-soc-lab
Username	sai
Source IP	127.0.0.1
Source Port	40092
Decoder	sshd
Rule ID	5715
Rule Level	3
Rule Description	sshd: authentication success.
MITRE IDs	T1078, T1021
MITRE Techniques	Valid Accounts, Remote Services

The Wazuh event contained:

Accepted password for sai from 127.0.0.1 port 40092 ssh2
Evidence — Successful SSH Authentication

## Step 4 — Investigate the Sudo Activity

The sudo whoami activity was also collected by Wazuh.

Wazuh Sudo Alert Details
Field	Result
Agent	kali-soc-lab
Current User	sai
Target User	root
Command	/usr/bin/whoami
Working Directory	/home/sai
Decoder	sudo
Rule ID	5402
Rule Level	3
Rule Description	Successful sudo to ROOT executed.
MITRE ID	T1548.003
MITRE Tactic	Privilege Escalation, Defense Evasion
MITRE Technique	Sudo and Sudo Caching

The important log information was:

USER=root ; COMMAND=/usr/bin/whoami

This confirmed that the user sai executed the command with root privileges.

Evidence — Sudo Activity

## Step 5 — Verify the Activity in Linux Logs

I correlated the Wazuh alerts with the original Linux systemd-journald logs.

The following command was used:

sudo journalctl --since "1 hour ago" --no-pager | grep -Ei "sshd|sudo"

The logs showed:

Aug 17 00:27:48 vbox sshd-session[3972]: Accepted password for sai from 127.0.0.1 port 40092 ssh2

followed later by:

Aug 17 00:31:43 vbox sudo[4046]: sai : TTY=pts/1 ; PWD=/home/sai ; USER=root ; COMMAND=/usr/bin/whoami

This provided local evidence that the successful SSH authentication was followed by privileged activity.

Evidence — Linux Correlation

## SOC L1 Investigation
1. Who successfully logged in?

The authenticated user was:

sai
2. Where did the login originate?

The source IP was:

127.0.0.1

This is the loopback address of the Kali Linux machine.

3. Was the SSH authentication successful?

Yes.

Wazuh Rule 5715 detected:

sshd: authentication success.
4. What happened after authentication?

The user executed:

sudo whoami

The command returned:

root
5. Was privileged activity detected?

Yes.

Wazuh Rule 5402 detected:

Successful sudo to ROOT executed.
6. Did the SSH login lead to root privileges?

The user obtained root privileges for the specific command executed through sudo.

The evidence was:

USER=root
COMMAND=/usr/bin/whoami

This does not mean the user permanently changed their account to root. It means the command was executed with root privileges through sudo.

## Event Correlation

The two Wazuh events were correlated as follows:

Successful SSH Authentication
        ↓
User: sai
        ↓
Source: 127.0.0.1
        ↓
SSH session established
        ↓
sudo whoami
        ↓
USER=root
        ↓
Wazuh Rule 5402
        ↓
Successful privileged activity

This demonstrates an important SOC L1 investigation principle:

Authentication events should be correlated with post-authentication activity to understand what happened after a user gained access.

## MITRE ATT&CK Mapping
Successful SSH Authentication
MITRE Information	Result
Technique ID	T1078
Technique	Valid Accounts
Technique ID	T1021
Technique	Remote Services
Sudo Activity
MITRE Information	Result
Technique ID	T1548.003
Technique	Sudo and Sudo Caching
Tactic	Privilege Escalation / Defense Evasion

## Alert Classification
Detection Classification

True Positive

Wazuh correctly detected the successful SSH authentication and the subsequent sudo activity.

Activity Classification

Benign / Authorized Security Test

The activity was intentionally generated by me in my own Kali Linux SOC laboratory environment.

Therefore, the activity was not a real unauthorized compromise.

Correct Wazuh Detection
        +
Authorized Lab Activity
        =
Benign Security Test

## SOC L1 Investigation Workflow
Successful SSH Login
        ↓
Identify User
        ↓
Identify Source IP
        ↓
Check Authentication Result
        ↓
Investigate Post-Login Activity
        ↓
Identify Sudo / Privileged Activity
        ↓
Correlate Events
        ↓
Determine Context
        ↓
Classify Activity

## SOC L1 Skills Practiced

Through this lab, I practiced:

Successful SSH authentication analysis
Linux authentication log analysis
systemd-journald investigation
Wazuh SIEM investigation
Authentication event analysis
Post-authentication investigation
Sudo activity analysis
Privileged activity detection
Event correlation
Source IP identification
Username identification
Command analysis
MITRE ATT&CK mapping
True positive detection analysis
SOC L1 alert classification

## Key Takeaways
A successful SSH login is not automatically malicious.
SOC analysts must investigate what happened after authentication.
Wazuh Rule 5715 detected the successful SSH authentication.
Wazuh Rule 5402 detected successful sudo-to-root activity.
The source IP, username, timestamp, and executed command are important investigation fields.
Correlating authentication events with subsequent activity provides better context.
sudo allows an authorized user to execute specific commands with elevated privileges.
A true positive detection does not necessarily mean a security incident occurred.
Alert classification must consider the context of the activity.

## Conclusion

This lab demonstrated the investigation of a successful SSH authentication followed by privileged activity.

A successful SSH login was performed using the sai account from 127.0.0.1. Wazuh detected the successful authentication using Rule 5715 at Level 3.

After authentication, the user executed sudo whoami, which returned root. Wazuh detected this activity using Rule 5402 at Level 3, identifying it as successful sudo-to-root activity.

The events were correlated with the Linux systemd-journald logs, confirming that the successful SSH authentication was followed by the privileged command execution.

Because the activity was intentionally performed within my own SOC laboratory, it was classified as a benign authorized security test.

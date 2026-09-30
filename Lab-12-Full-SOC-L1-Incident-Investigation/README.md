# Lab 12 — Full SOC L1 Incident Investigation & Response

## Objective

Perform a complete SOC L1 incident investigation using Wazuh by simulating a controlled SSH-based attack sequence and following the complete security operations workflow:

- Detect suspicious authentication activity
- Identify brute-force behavior
- Investigate successful authentication
- Investigate privileged activity
- Create an investigation artifact
- Build an incident timeline
- Perform containment
- Perform recovery
- Clean up the lab environment

This lab demonstrates how a SOC L1 analyst can move from **initial detection → investigation → correlation → containment → recovery** using Wazuh.

---

## Lab Environment

| Component | Details |
|---|---|
| SIEM / XDR | Wazuh |
| Wazuh Server | Ubuntu Server |
| Wazuh Agent | Kali Linux |
| Agent Name | `kali-soc-lab` |
| Attacker / Test System | Kali Linux |
| Target | Kali Linux SSH service |
| Network | VirtualBox Host-Only Network |
| Monitoring | Wazuh Dashboard |
| Log Source | Linux journald / SSH logs |

---

## Lab Architecture

```text
                 ┌─────────────────────────┐
                 │      Kali Linux         │
                 │     Wazuh Agent         │
                 │                         │
                 │  SSH Authentication     │
                 │  Privileged Activity    │
                 └────────────┬────────────┘
                              │
                              │ Security Events
                              ▼
                 ┌─────────────────────────┐
                 │     Wazuh Server        │
                 │                         │
                 │  Wazuh Manager         │
                 │  Wazuh Indexer         │
                 │  Wazuh Dashboard       │
                 └────────────┬────────────┘
                              │
                              ▼
                 ┌─────────────────────────┐
                 │     SOC Investigation   │
                 │                         │
                 │ Detection               │
                 │ Correlation             │
                 │ Investigation           │
                 │ Containment             │
                 │ Recovery                │
                 └─────────────────────────┘

## Incident Scenario

A controlled SSH attack sequence was simulated on the Kali Linux system.

The simulated activity followed this sequence:

Multiple Failed SSH Attempts
          ↓
Brute-Force Detection
          ↓
Successful SSH Authentication
          ↓
Privileged Sudo Activity
          ↓
Investigation Artifact Creation
          ↓
Containment — Stop SSH
          ↓
Recovery — Restart SSH
          ↓
Artifact Cleanup

The activity was performed entirely inside the isolated home lab environment.

## Step 1 — Generate Suspicious SSH Activity

Multiple SSH authentication attempts were generated using a non-existent user.

for i in {1..5}; do ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no -o StrictHostKeyChecking=no wronguser@127.0.0.1; done

The attempts generated authentication failures that were collected by the Wazuh agent.

Observed Wazuh Rules
Rule ID	Level	Description
5710	5	sshd: Attempt to login using a non-existent user
2502	10	syslog: User missed the password more than one time
5712	10	sshd: brute force trying to get access to the system. Non existent user.

These events provided the initial indication of suspicious authentication activity.

Evidence

Screenshot:

screenshots/01-initial-ssh-detection.png

## Step 2 — Detection and Initial Triage

The Wazuh Dashboard was used as the primary SOC investigation interface.

The initial events showed repeated SSH authentication failures involving a non-existent account.

The analyst reviewed:

Event timestamps
Agent name
Rule IDs
Rule severity
Authentication activity
Repeated failed attempts
Brute-force detection

The combination of repeated authentication failures and the Wazuh brute-force rule provided the initial incident indicator.

## Step 3 — Successful SSH Authentication

After the failed authentication sequence, a successful SSH login was performed using the authorized account.

ssh sai@127.0.0.1

Wazuh detected the successful authentication.

Observed Rule
Rule ID	Level	Description
5715	3	sshd: authentication success.

This event was important because it changed the investigation from isolated authentication failures to a sequence containing a successful login.

Evidence

Screenshot:

screenshots/02-successful-authentication.png

## Step 4 — Privileged Activity Investigation

After successful authentication, privileged activity was performed.

sudo whoami

Output:

root

Wazuh detected the successful privilege escalation through sudo.

Observed Rule
Rule ID	Level	Description
5402	3	Successful sudo to ROOT executed.

The event contained important investigation fields including:

srcuser = sai
dstuser = root
command = /usr/bin/whoami
location = journald
MITRE ATT&CK
T1548.003 — Sudo and Sudo Caching
Evidence

Screenshot:

screenshots/03-privileged-activity.png

## Step 5 — Investigation Artifact

A temporary investigation artifact was created using privileged access.

sudo touch /tmp/lab12-investigation-artifact

The command generated a Wazuh sudo event showing:

command = /usr/bin/touch /tmp/lab12-investigation-artifact
srcuser = sai
dstuser = root

The artifact creation was therefore evidenced through the privileged command execution event.

Important Note

Wazuh did not display a separate File Integrity Monitoring event for this specific artifact during this lab.

Therefore, this lab does not claim that FIM detected the artifact.

The artifact is documented as evidence of privileged command execution.

## Step 6 — Incident Timeline

The Wazuh Events interface was used to correlate the events chronologically.

The investigation timeline contained:

09:57–09:59
Multiple failed SSH authentication attempts
        ↓
Rule 5710
Rule 2502
Rule 5712
        ↓
10:02
Successful SSH authentication
Rule 5715
        ↓
10:05
Privileged sudo activity
Rule 5402
        ↓
Investigation artifact creation
        ↓
Containment
        ↓
Recovery

The Wazuh Events view was filtered using:

rule.id:(5710 OR 2502 OR 5712 OR 5715 OR 5402)

This allowed the relevant authentication and privileged activity events to be reviewed together.

Evidence

Screenshot:

screenshots/04-incident-timeline.png

## Step 7 — Containment

After the suspicious activity sequence was investigated, SSH access was temporarily stopped as the containment action.

sudo systemctl stop ssh

The SSH service status was checked to confirm that the service had stopped.

The purpose of this action was to simulate a SOC containment decision in which remote SSH access is temporarily restricted while an investigation is performed.

## Step 8 — Recovery

After containment, the SSH service was restored.

sudo systemctl start ssh

The service status was checked and confirmed to be active and running.

This simulated the recovery stage of the incident response process.

## Step 9 — Cleanup

The temporary investigation artifact was removed.

sudo rm /tmp/lab12-investigation-artifact

No output indicated that the cleanup command completed successfully.

The lab environment was returned to its normal state.

## SOC L1 Investigation

Initial Detection

The investigation began with multiple failed SSH authentication attempts.

## The analyst identified:

Repeated authentication failures
Non-existent SSH username
Multiple related Wazuh alerts
Brute-force detection
Investigation

## The analyst then correlated the authentication events with later activity.

The sequence showed:

Failed Authentication
        ↓
Brute-Force Detection
        ↓
Successful Authentication
        ↓
Privileged Sudo Activity

This chronological relationship provided a stronger investigation context than analyzing each alert independently.

## Privilege Investigation

The successful sudo event was investigated using:

Rule ID: 5402
Level: 3
Source User: sai
Destination User: root
Command: /usr/bin/whoami
Location: journald

## The event was mapped to:

MITRE ATT&CK T1548.003
Sudo and Sudo Caching

## Containment

The SSH service was stopped:

sudo systemctl stop ssh

This simulated restricting further SSH access while investigating the incident.

## Recovery

The SSH service was restored:

sudo systemctl start ssh

The service was verified as active and running.

## Incident Classification

Category	Classification
Incident Type	SSH Authentication / Brute-Force Activity
Initial Detection	Failed SSH Authentication
Detection Source	Wazuh
Authentication Activity	Multiple Failed Attempts
Brute-Force Detection	Rule 5712
Successful Authentication	Rule 5715
Privileged Activity	Rule 5402
Privilege Technique	T1548.003
Containment	SSH Service Stopped
Recovery	SSH Service Restarted
Environment	Controlled Home Lab

This was a controlled security simulation performed for SOC investigation practice.

## MITRE ATT&CK Mapping

Activity	Wazuh Rule	MITRE ATT&CK
SSH Brute-Force Activity	5712	Credential Access / Brute Force
Successful SSH Authentication	5715	Authentication Activity
Privileged Sudo Execution	5402	T1548.003 — Sudo and Sudo Caching

The MITRE mapping shown for the privileged sudo event was directly provided by the Wazuh alert.

## SOC L1 Investigation Workflow

1. Alert Detection
       ↓
2. Initial Triage
       ↓
3. Validate Authentication Events
       ↓
4. Identify Brute-Force Activity
       ↓
5. Investigate Successful Login
       ↓
6. Investigate Privileged Activity
       ↓
7. Correlate Events into Timeline
       ↓
8. Identify Investigation Artifact
       ↓
9. Containment
       ↓
10. Recovery
       ↓
11. Cleanup
       ↓
12. Document Findings

## Evidence Summary

Evidence	Result
Failed SSH Attempts	Detected
Non-existent SSH User	Detected
Repeated Password Failures	Detected
SSH Brute-Force Detection	Detected
Successful SSH Login	Detected
Sudo to Root	Detected
Privileged Command Execution	Detected
Investigation Artifact	Created
FIM Alert for Artifact	Not observed
Incident Timeline	Correlated in Wazuh
SSH Containment	Performed
SSH Recovery	Performed
Artifact Cleanup	Completed

##Screenshots

screenshots/
│
├── 01-initial-ssh-detection.png
├── 02-successful-authentication.png
├── 03-privileged-activity.png
└── 04-incident-timeline.png
Screenshot 1 — Initial SSH Detection

Shows multiple failed SSH authentication attempts and Wazuh brute-force related alerts.

Screenshot 2 — Successful Authentication

Shows the successful SSH authentication event detected by Wazuh Rule 5715.

Screenshot 3 — Privileged Activity

Shows the successful sudo-to-root event detected by Wazuh Rule 5402.

Screenshot 4 — Incident Timeline

Shows the correlated authentication and privileged activity events in the Wazuh Events interface.

##Skills Demonstrated

SOC L1 alert triage
Security event investigation
SSH log analysis
Authentication monitoring
Brute-force detection
Wazuh SIEM investigation
Event correlation
Incident timeline construction
Privileged activity investigation
Linux security monitoring
MITRE ATT&CK mapping
Incident containment
Incident recovery
Evidence documentation
Security incident response workflow

##Key Takeaways

Multiple authentication failures can provide an early indication of brute-force activity.
Wazuh can correlate different Linux security events through its rule engine.
Successful authentication after repeated failures requires additional investigation.
Privileged sudo activity provides important context during incident investigation.
A SOC analyst should investigate events chronologically rather than viewing alerts in isolation.
Wazuh Dashboard provides the primary investigation interface for reviewing and correlating security events.
Containment and recovery are important stages of the SOC incident response process.
Investigation evidence should be documented accurately without claiming detections that were not observed.

## Conclusion

This lab demonstrated a complete SOC L1 investigation workflow using Wazuh.

A controlled SSH attack sequence was generated and investigated from the initial failed authentication attempts through brute-force detection, successful authentication, privileged activity, artifact creation, containment, recovery, and cleanup.

The lab demonstrates practical SOC capabilities including:

Detection
   ↓
Triage
   ↓
Investigation
   ↓
Correlation
   ↓
MITRE Mapping
   ↓
Containment
   ↓
Recovery
   ↓
Documentation

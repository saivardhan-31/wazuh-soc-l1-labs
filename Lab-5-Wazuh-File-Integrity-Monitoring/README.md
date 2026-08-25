# Lab 5 — File Integrity Monitoring (FIM) using Wazuh

## Objective

The objective of this lab was to demonstrate File Integrity Monitoring (FIM) using Wazuh by monitoring a file on a Linux endpoint and detecting when the file was created, modified, and deleted.

This lab helped me practice:

- File Integrity Monitoring
- Linux file monitoring
- Wazuh Syscheck analysis
- Real-time file change detection
- File creation detection
- File modification detection
- File deletion detection
- Hash-based integrity analysis
- Wazuh alert investigation
- Rule and severity analysis
- MITRE ATT&CK mapping
- Basic SOC L1 alert triage

---

## Lab Environment

| **Component** | **Details** |
|---|---|
| SIEM | Wazuh Cloud |
| Endpoint | Kali Linux 2025.4 |
| Architecture | x86_64 |
| Wazuh Agent | `kali-soc-lab` |
| Wazuh Version | 4.14.7 |
| Monitoring Module | Syscheck / FIM |
| Monitored Directory | `/etc` |
| Test File | `/etc/wazuh-fim-test.txt` |

---

## Lab Architecture
Kali Linux
    |
    | File creation / modification / deletion
    ↓
Wazuh Syscheck
    |
    | File Integrity Monitoring
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

## Scenario

A controlled file integrity monitoring test was performed on my own Kali Linux SOC laboratory environment.

The purpose was to verify that Wazuh could:

Monitor files on the Linux endpoint
Detect a newly created file
Detect changes to an existing file
Detect file deletion
Generate Wazuh alerts
Record file metadata and integrity hashes
Provide information useful for SOC L1 investigation
The test was performed only against my own lab environment.
## Step 1 — Verify Wazuh FIM Configuration

I verified that the Wazuh File Integrity Monitoring configuration was enabled:

sudo grep -n -A5 -B2 "<syscheck>" /var/ossec/etc/ossec.conf

The output showed:

<syscheck>
    <disabled>no</disabled>

This confirmed that the Wazuh Syscheck module was enabled.

The configured monitoring directories included:

/etc
/usr/bin
/usr/sbin
/bin
/sbin
/boot

I verified the monitored directories using:

sudo grep -nE "<directories|<realtime|<whodata|</directories>" /var/ossec/etc/ossec.conf

The output confirmed that /etc was being monitored.

## Step 2 — Create a Test File

I created a test file inside the monitored /etc directory:

sudo touch /etc/wazuh-fim-test.txt

I then verified that the file was created:

ls -l /etc/wazuh-fim-test.txt

The file was successfully created with root ownership and a size of 0 bytes.

## Step 3 — Restart Wazuh Agent

To ensure that the FIM configuration was actively running, I restarted the Wazuh Agent:

sudo systemctl restart wazuh-agent

I verified the service status:

sudo systemctl status wazuh-agent

The output showed:

Active: active (running)

The Wazuh Syscheck process was also running:

/var/ossec/bin/wazuh-syscheckd

This confirmed that the File Integrity Monitoring component was active.

## Step 4 — Investigate File Creation Alert

The file creation event was investigated in:

Wazuh Dashboard → Threat Hunting → Events

The alert showed:

Field	Result
Agent	kali-soc-lab
Decoder	syscheck_new_entry
Location	syscheck
File	/etc/wazuh-fim-test.txt
Event	added
Mode	realtime
Rule Description	File added to the system
Rule Level	5
Rule ID	554

The Wazuh event showed:

File '/etc/wazuh-fim-test.txt' added
Mode: realtime

This confirmed that Wazuh detected the creation of the file in real time.

Evidence

## Step 5 — Modify the File

Next, I modified the contents of the monitored file:

echo "Wazuh FIM test" | sudo tee /etc/wazuh-fim-test.txt

The modification changed the file's size and integrity hashes.

Wazuh detected the modification and generated another alert.

## Step 6 — Investigate File Modification Alert

The modification event was investigated in:

Wazuh Dashboard → Threat Hunting → Events

The event showed:

Field	Result
Agent	kali-soc-lab
Decoder	syscheck_integrity_changed
Location	syscheck
File	/etc/wazuh-fim-test.txt
Event	modified
Mode	realtime
Rule Description	Integrity checksum changed
Rule Level	7
Rule ID	550

The alert showed:

File '/etc/wazuh-fim-test.txt' modified
Mode: realtime

Wazuh also recorded changes to file attributes including:

size
mtime
md5
sha1
sha256

This demonstrates that Wazuh can compare file integrity information before and after a modification.

Evidence

## Step 7 — Delete the File

Finally, I deleted the test file:

sudo rm /etc/wazuh-fim-test.txt

The file was intentionally deleted as part of the controlled FIM test.

## Step 8 — Investigate File Deletion Alert

The deletion event was investigated in:

Wazuh Dashboard → Threat Hunting → Events

The alert showed:

Field	Result
Agent	kali-soc-lab
Decoder	syscheck_deleted
Location	syscheck
File	/etc/wazuh-fim-test.txt
Event	deleted
Mode	realtime
Rule Description	File deleted
Rule Level	7
Rule ID	553

The event showed:

File '/etc/wazuh-fim-test.txt' deleted
Mode: realtime

This confirmed that Wazuh successfully detected the deletion of the monitored file.

## Evidence

FIM Detection Summary
Action	Wazuh Detection	Rule	Level
File created	File added to the system	554	5
File modified	Integrity checksum changed	550	7
File deleted	File deleted	553	7

The complete detection lifecycle was:

File Created
     ↓
Wazuh Detects Addition
     ↓
File Modified
     ↓
Wazuh Detects Integrity Change
     ↓
File Deleted
     ↓
Wazuh Detects Deletion
Alert Analysis
File Creation

Wazuh detected:

File '/etc/wazuh-fim-test.txt' added

This indicates that a new file appeared inside a monitored directory.

## A SOC analyst would investigate:

Who created the file?
When was it created?
What is the file's purpose?
What process created it?
Is the file expected?
Does the file contain suspicious content?

## File Modification

Wazuh detected:

Integrity checksum changed

The alert included changes to file attributes and integrity hashes.

A modification to a sensitive system file could potentially indicate:

Unauthorized configuration changes
Malware activity
Persistence
Tampering
Privilege escalation

In this lab, the modification was intentional and controlled.

## File Deletion

Wazuh detected:

File '/etc/wazuh-fim-test.txt' deleted

File deletion can be important during incident investigations because attackers may attempt to remove:

Configuration files
Malware artifacts
Scripts
Logs
Evidence

In this laboratory exercise, the deletion was intentionally performed as part of the FIM test.

## MITRE ATT&CK Mapping

The Wazuh alerts provided MITRE ATT&CK mappings related to file and data manipulation.

For the modification event, Wazuh identified:

Tactic: Impact
Technique: Stored Data Manipulation

For the deletion event, Wazuh identified:

Tactic: Defense Evasion / Impact
Technique: File Deletion / Data Destruction

These mappings help a SOC analyst understand how file integrity events may relate to attacker behavior.

## SOC L1 Investigation
What happened?

A file was created, modified, and deleted inside a directory monitored by Wazuh FIM.

Which file was monitored?
/etc/wazuh-fim-test.txt
Which Wazuh module detected the activity?
Syscheck / File Integrity Monitoring
Was the file creation detected?

Yes.

Rule ID: 554
Level: 5
Was the file modification detected?

Yes.

Rule ID: 550
Level: 7
Was the file deletion detected?

Yes.

Rule ID: 553
Level: 7
Was this malicious activity?

No.

The activity was intentionally generated as part of the controlled SOC laboratory exercise.

## Alert Classification

Classification: Benign / Controlled Test

The file was intentionally created, modified, and deleted on my own Kali Linux SOC laboratory endpoint.

There was no unauthorized access or malicious activity.

The purpose was to validate the Wazuh FIM detection pipeline.

## SOC L1 Skills Practiced

Through this lab, I practiced:

File Integrity Monitoring
Linux file monitoring
Wazuh Syscheck investigation
Real-time detection
File creation analysis
File modification analysis
File deletion analysis
File hash comparison
Wazuh rule ID analysis
Alert severity analysis
File path identification
MITRE ATT&CK mapping
SOC L1 alert triage
Benign vs suspicious activity classification

## Investigation Workflow
File Created
     ↓
Kali Linux
     ↓
Wazuh Syscheck
     ↓
Wazuh Agent
     ↓
Wazuh Cloud
     ↓
Wazuh Alert
     ↓
SOC L1 Investigation
     ↓
File Modified
     ↓
Integrity Change Detected
     ↓
File Deleted
     ↓
Deletion Detected
     ↓
Benign Controlled Test

## Key Takeaways

File Integrity Monitoring helps detect unauthorized changes to important files and directories.

Wazuh Syscheck successfully monitored the /etc directory on the Kali Linux endpoint.

Wazuh detected the creation of /etc/wazuh-fim-test.txt in real time.

Wazuh detected changes to the file's size, modification time, and integrity hashes.

Wazuh also detected the deletion of the file.

The alerts demonstrated how FIM can provide valuable evidence for SOC analysts investigating potential system tampering.

A file integrity alert does not automatically mean malicious activity. A SOC analyst must investigate the context, affected file, user, process, timing, and surrounding events before classifying the alert.

## Conclusion

This lab successfully demonstrated File Integrity Monitoring using Wazuh.

A test file was created inside the monitored /etc directory, modified, and subsequently deleted.

Wazuh successfully detected all three activities:

File Creation      → Rule 554
File Modification  → Rule 550
File Deletion      → Rule 553

The events were successfully collected from Kali Linux by the Wazuh Agent and displayed in Wazuh Cloud.

The activity was classified as a benign controlled test because all actions were intentionally performed within my own SOC practice environment.

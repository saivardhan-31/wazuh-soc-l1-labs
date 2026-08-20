# Lab 4 — Listening Port Change Detection using Wazuh

## Objective

The objective of this lab was to detect and investigate a new listening network port on a Kali Linux endpoint using Wazuh.

In this lab, I created a controlled Python HTTP server on localhost, verified the newly opened listening port, investigated the Wazuh detection, identified the process responsible for opening the port, and verified the service locally.

This lab helped me practice:

- Network listening-port investigation
- Wazuh port-change detection
- Linux socket investigation
- Process identification
- Network service verification
- Event investigation
- SOC L1 alert triage
- Alert classification

---

## Lab Environment

| Component | Details |
|---|---|
| SIEM | Wazuh Cloud |
| Endpoint | Kali Linux |
| Wazuh Agent | `kali-soc-lab` |
| Network Service | Python HTTP Server |
| Test Port | `8080` |
| Bind Address | `127.0.0.1` |
| Process | `python3` |

---

## Lab Architecture

Kali Linux
    |
    | Start Python HTTP Server
    ↓
127.0.0.1:8080
    |
    ↓
Listening Port Change
    |
    ↓
Wazuh Agent
    |
    ↓
Wazuh Cloud
    |
    ↓
Rule 533
    |
    ↓
SOC L1 Investigation
    |
    ↓
Identify Process
    |
    ↓
Verify Service
    |
    ↓
Classify Activity

## Scenario

A controlled network service was started on my own Kali Linux laboratory machine.

A Python HTTP server was configured to listen on:

127.0.0.1:8080

The purpose was to generate a listening-port change and investigate how Wazuh detects and reports changes to listening network ports.

The activity was performed only within my own laboratory environment.

## Step 1 — Start a Test HTTP Server

A temporary Python HTTP server was started using:

python3 -m http.server 8080 --bind 127.0.0.1

The server created a new listening service on:

127.0.0.1:8080

The server was bound to the localhost interface so that it was accessible only from the Kali machine.

## Step 2 — Verify the Listening Port

The listening port was verified using:

sudo ss -lntp | grep :8080

The output showed:

LISTEN 0 5 127.0.0.1:8080 0.0.0.0:* users:(("python3",pid=5199,fd=3))

This confirmed:

Field	Result
IP Address	127.0.0.1
Port	8080
State	LISTEN
Process	python3
PID	5199
Evidence

## Step 3 — Detect the Port Change in Wazuh

The Wazuh dashboard was checked under:

Threat Hunting → Events

The Wazuh agent was filtered using:

agent.name: kali-soc-lab

Wazuh detected a change in the system's listening-port state.

Wazuh Alert Details
Field	Result
Agent	kali-soc-lab
Decoder	ossec
Location	netstat listening ports
Rule ID	533
Rule Level	7
Rule Description	Listened ports status (netstat) changed (new port opened or closed).
Rule Group	ossec

The alert indicated that the listening-port state had changed because a new port was opened.

Evidence

## Step 4 — Identify the Process Responsible

After Wazuh detected the port change, the process responsible for opening the port was investigated.

The following command was used:

sudo ss -lntp | grep python

The result showed:

LISTEN 0 5 127.0.0.1:8080 0.0.0.0:* users:(("python3",pid=5199,fd=3))

The process was then investigated using:

ps -fp $(pgrep -f "http.server 8080")

The result showed:

UID   PID   PPID   C   STIME   TTY   TIME   CMD
sai   5199  5163   0   17:20   pts/0 00:00:01 python3 -m http.server 8080 --bind 127.0.0.1

This confirmed that:

Process: python3
PID: 5199
Command: python3 -m http.server 8080
Bind address: 127.0.0.1

## Step 5 — Verify the HTTP Service

The service was tested locally using:

curl http://127.0.0.1:8080

The server returned an HTTP directory listing beginning with:

<!DOCTYPE HTML>
<html lang="en">

This confirmed that the service was actively responding on port 8080.

Evidence

## Step 6 — Stop the Test Service

After completing the investigation, the Python process was stopped.

The process was terminated using its PID:

kill 5199

The listening-port state was then verified again:

sudo ss -lntp | grep :8080

No output was returned.

This confirmed that:

Python server stopped
        ↓
Port 8080 closed
        ↓
Listening-port change reverted

## SOC L1 Investigation
1. What changed?

A new listening network port appeared on the Kali Linux endpoint.

2. Which port was opened?
8080
3. Which IP address was used?
127.0.0.1

This is the localhost/loopback address.

4. Which process opened the port?
python3
5. What was the process ID?
5199
6. What command started the service?
python3 -m http.server 8080 --bind 127.0.0.1
7. Did Wazuh detect the change?

Yes.

Wazuh generated:

Rule ID: 533
Level: 7

with the description:

Listened ports status (netstat) changed (new port opened or closed).
8. Was the service accessible?

Yes.

The service responded successfully to:

curl http://127.0.0.1:8080
9. Was the activity malicious?

No.

The port was intentionally opened as part of a controlled SOC laboratory exercise.

Event Investigation Workflow
Python HTTP Server Started
        ↓
127.0.0.1:8080 Listening
        ↓
ss Confirms Listening Port
        ↓
Wazuh Detects Port Change
        ↓
Rule 533 / Level 7
        ↓
Identify Process
        ↓
python3 / PID 5199
        ↓
Verify HTTP Service
        ↓
curl Successful
        ↓
Stop Test Service
        ↓
Port 8080 Closed

## Alert Classification
Detection Classification

True Positive

Wazuh correctly detected a change in the system's listening-port state.

## Activity Classification

Benign / Authorized Security Test

The new listening service was intentionally created by me on my own Kali Linux laboratory machine.

The service was bound specifically to:

127.0.0.1

and was stopped after the investigation.

Therefore, there was no confirmed malicious activity.

## MITRE ATT&CK Context

A newly opened listening port can be relevant during security investigations because attackers may establish network services or backdoors after compromising a system.

However, the Wazuh event in this lab was specifically a port-change detection, and the activity itself was an authorized laboratory test.

The investigation therefore focused on identifying:

The port
The process
The command
The network binding
The purpose of the service

## SOC L1 Skills Practiced

Through this lab, I practiced:

Listening-port monitoring
Linux ss command
Network service investigation
Process identification
PID investigation
Wazuh event investigation
Wazuh rule analysis
Port-change detection
Local service verification
HTTP service testing
Event correlation
Alert classification
SOC L1 investigation methodology

## Key Takeaways

A newly opened listening port can be an important SOC investigation indicator.
Wazuh can detect changes in the system's listening-port state.
Wazuh Rule 533 detected the port change in this lab.
The affected port was 8080.
The service was bound to 127.0.0.1.
The process responsible was python3, PID 5199.
ss can be used to identify listening ports and associated processes.
ps can be used to investigate the process and command line.
curl can be used to verify whether a local HTTP service is responding.
SOC analysts should identify the process responsible for a new listening port rather than immediately assuming malicious activity.
Context is important when classifying security alerts.

## Conclusion

This lab demonstrated the detection and investigation of a new listening network port using Wazuh.

A temporary Python HTTP server was started on 127.0.0.1:8080. The listening port was verified using the Linux ss command.

Wazuh detected the change and generated Rule 533 at Level 7, reporting that the listened-port status had changed.

Further investigation identified the responsible process as python3, with PID 5199, running:

python3 -m http.server 8080 --bind 127.0.0.1

The service was successfully verified using curl.

After completing the investigation, the Python process was stopped and port 8080 was confirmed to be no longer listening.

The activity was classified as a true positive detection of a port-state change, but the underlying activity was classified as a benign authorized security test.

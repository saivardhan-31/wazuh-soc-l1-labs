# Lab 8 — Web Attack Detection using Wazuh

## Objective

Detect and investigate a simulated SQL injection attack against an Apache web server using Wazuh.

The lab demonstrates Apache access-log collection, Wazuh web-log decoding, SQL injection rule matching, alert generation, and SOC L1 investigation.

---

## Lab Environment

- **Attacker / Web Server:** Kali Linux
- **Web Server:** Apache2
- **SIEM:** Wazuh
- **Wazuh Agent:** kali-soc-lab
- **Wazuh Server:** Ubuntu Server
- **Log Source:** `/var/log/apache2/access.log`
- **Attack Type:** SQL Injection
- **Tool:** curl

---

## Lab Architecture

Kali Linux
   │
   │ SQL Injection Request
   ▼
Apache Web Server
   │
   │ access.log
   ▼
Wazuh Agent
   │
   │ Log Forwarding
   ▼
Wazuh Server
   │
   │ Rule 31164
   ▼
Wazuh Dashboard
   │
   ▼
SOC L1 Investigation

## Scenario

A controlled SQL injection request was sent to the Apache web server.

The request used:

## curl "http://127.0.0.1/?id=%27"

Apache recorded the request in its access log.

## Step 1 — Verify Apache

Apache was confirmed to be running:

sudo systemctl status apache2 --no-pager

Status:

Active: active (running)
## Step 2 — Configure Apache Log Collection

Wazuh Agent was configured to monitor the Apache access log:

<localfile>
    <log_format>apache</log_format>
    <location>/var/log/apache2/access.log</location>
</localfile>

The Wazuh Agent was restarted after configuration verification.

## Step 3 — Verify Wazuh Log Collection

The Apache access log was confirmed to be actively monitored by the Wazuh log collector.

sudo lsof /var/log/apache2/access.log

The output showed:

wazuh-log ... /var/log/apache2/access.log

This confirmed that Wazuh was actively reading the Apache access log.

## Step 4 — Simulate SQL Injection

A controlled SQL injection-style request was generated:

curl "http://127.0.0.1/?id=%27"

Apache successfully processed the request and returned its web page.

## Step 5 — Verify Apache Log

The request was recorded in Apache:

127.0.0.1 - - [26/Sep/2026:10:25:31 +0530] "GET /?id=%27 HTTP/1.1" 200 10958 "-" "curl/8.18.0"
## Step 6 — Validate Wazuh Detection Rule

The Apache log was tested using Wazuh wazuh-logtest.

The event was decoded as:

Decoder: web-accesslog

The installed Wazuh ruleset matched:

Rule ID: 31164
Level: 6
Description: SQL injection attempt.

The rule also contained MITRE ATT&CK mappings:

T1055
T1190
## Step 7 — Wazuh Alert

The actual event generated a Wazuh alert.

## Alert Details
Field	Value
Agent	kali-soc-lab
Source IP	127.0.0.1
Protocol	GET
URL	/?id=%27
Decoder	web-accesslog
Log Source	/var/log/apache2/access.log
Rule ID	31164
Level	6
Description	SQL injection attempt
MITRE ATT&CK	T1055, T1190
Alert Analysis

## The Wazuh alert confirms that:

Apache received the HTTP request.
The request was written to access.log.
Wazuh Agent collected the log.
Wazuh decoded it using web-accesslog.
Rule 31164 identified the event as a SQL injection attempt.
A Level 6 Wazuh alert was generated.
## MITRE ATT&CK Mapping

The Wazuh rule reported the following MITRE ATT&CK mappings:

T1055 — Process Injection
T1190 — Exploit Public-Facing Application

These mappings are provided by the Wazuh rule associated with the alert.

## SOC L1 Investigation
Initial Triage
Alert: SQL injection attempt
Severity: Level 6
Source: Apache access log
Source IP: 127.0.0.1
Requested URL: /?id=%27
Affected Service: Apache HTTP Server
## Investigation Steps
Review the Wazuh alert.
Identify the source IP.
Examine the requested URL.
Verify the original Apache access-log entry.
Check the Wazuh rule and decoder.
Determine whether the activity was authorized or malicious.
Document the event and evidence.
## Classification
Category: Web Attack
Attack Type: SQL Injection
Detection Source: Apache access logs
Wazuh Rule: 31164
Alert Level: 6
Environment: Controlled SOC lab

The activity was intentionally generated for security monitoring and detection testing.

## SOC Workflow
Web Request
     ↓
Apache Access Log
     ↓
Wazuh Agent
     ↓
Wazuh Manager
     ↓
Web Access Decoder
     ↓
Rule 31164
     ↓
SQL Injection Alert
     ↓
SOC L1 Triage
     ↓
Investigation & Classification
## Skills Practiced
Apache log monitoring
Web attack detection
SQL injection detection
Wazuh rule testing
Wazuh log decoding
Log analysis
Alert investigation
MITRE ATT&CK mapping
SOC L1 alert triage
Incident classification
## Key Takeaways
Apache access logs provide valuable web security telemetry.
Wazuh can decode Apache access logs using the web-accesslog decoder.
Wazuh Rule 31164 can identify specific SQL injection patterns.
wazuh-logtest is useful for validating detection rules before generating live events.
SOC analysts should verify the original log source when investigating an alert.
## Conclusion

This lab successfully demonstrated end-to-end SQL injection detection using Apache and Wazuh.

The simulated request was logged by Apache, collected by the Wazuh Agent, decoded by Wazuh, matched by Rule 31164, and displayed as a Level 6 SQL injection alert in the Wazuh Dashboard.

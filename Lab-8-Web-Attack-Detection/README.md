# Lab 8 — Web Attack Detection using Wazuh

## Objective

Detect and investigate a simulated SQL injection attack against an Apache web server using Wazuh.

The lab demonstrates Apache access-log collection, Wazuh web-log decoding, SQL injection rule matching, alert generation, and SOC L1 investigation.

## Lab Environment

- Kali Linux
- Apache2 Web Server
- Wazuh Agent
- Wazuh Server
- Wazuh Dashboard
- curl
- Linux Apache access logs

## Lab Architecture

```text
Kali Linux
    |
    | SQL Injection Request
    v
Apache Web Server
    |
    | access.log
    v
Wazuh Agent
    |
    | Log Forwarding
    v
Wazuh Server
    |
    | Rule 31164
    v
Wazuh Dashboard
    |
    v
SOC L1 Investigation

Scenario

A controlled SQL injection request was generated against the Apache web server.

The request used:

curl "http://127.0.0.1/?id=%27"

Apache recorded the request in its access log, and Wazuh detected the activity using its web attack ruleset.

Step 1 — Verify Apache

Apache was confirmed to be running:

sudo systemctl status apache2 --no-pager

Status:

Active: active (running)
Step 2 — Configure Apache Log Collection

The Wazuh Agent was configured to monitor the Apache access log:

<localfile>
    <log_format>apache</log_format>
    <location>/var/log/apache2/access.log</location>
</localfile>

The Wazuh Agent was restarted after verifying the configuration.

Step 3 — Verify Wazuh Log Collection

The Apache access log was confirmed to be actively monitored by the Wazuh log collector:

sudo lsof /var/log/apache2/access.log

The output showed the wazuh-log process reading:

/var/log/apache2/access.log

This confirmed that Wazuh was actively collecting Apache access logs.

Step 4 — Simulate SQL Injection

A controlled SQL injection request was generated:

curl "http://127.0.0.1/?id=%27"

Apache successfully processed the request and returned the default web page.

Step 5 — Verify Apache Log

The request was recorded in the Apache access log:

127.0.0.1 - - [26/Sep/2026:10:25:31 +0530] "GET /?id=%27 HTTP/1.1" 200 10958 "-" "curl/8.18.0"
Step 6 — Validate Wazuh Detection Rule

The Apache log was tested using:

sudo /var/ossec/bin/wazuh-logtest

Wazuh decoded the event using:

Decoder: web-accesslog

The installed Wazuh ruleset matched:

Rule ID: 31164
Level: 6
Description: SQL injection attempt.

The rule also reported MITRE ATT&CK mappings:

T1055
T1190
Step 7 — Wazuh Alert

The actual Apache request generated a Wazuh alert.

Alert Details
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

The alert confirms the complete detection pipeline:

Apache received the HTTP request.
Apache recorded the request in access.log.
Wazuh Agent collected the log.
Wazuh decoded the event using web-accesslog.
Rule 31164 identified the SQL injection pattern.
Wazuh generated a Level 6 alert.
The alert was displayed in the Wazuh Dashboard.
MITRE ATT&CK Mapping

The Wazuh rule reported:

T1055 — Process Injection
T1190 — Exploit Public-Facing Application

These mappings are provided by the Wazuh detection rule associated with the alert.

SOC L1 Investigation
Initial Triage
Alert: SQL injection attempt
Severity: Level 6
Source: Apache access log
Source IP: 127.0.0.1
Requested URL: /?id=%27
Affected Service: Apache HTTP Server
Investigation Steps
Review the Wazuh alert.
Identify the source IP.
Examine the requested URL.
Verify the original Apache access-log entry.
Review the Wazuh decoder and rule.
Determine whether the activity was authorized or malicious.
Document the evidence and investigation results.
Classification
Category: Web Attack
Attack Type: SQL Injection
Detection Source: Apache access logs
Wazuh Rule: 31164
Alert Level: 6
Environment: Controlled SOC Lab

The activity was intentionally generated for security monitoring and detection testing.

SOC Workflow
Web Request
     |
     v
Apache Access Log
     |
     v
Wazuh Agent
     |
     v
Wazuh Manager
     |
     v
Web Access Decoder
     |
     v
Rule 31164
     |
     v
SQL Injection Alert
     |
     v
SOC L1 Triage
     |
     v
Investigation & Classification
Skills Practiced
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
Key Takeaways
Apache access logs provide useful web security telemetry.
Wazuh can decode Apache access logs using the web-accesslog decoder.
Wazuh Rule 31164 can identify specific SQL injection patterns.
wazuh-logtest can be used to validate detection rules before generating live events.
SOC analysts should verify the original log source during alert investigation.
Conclusion

This lab successfully demonstrated end-to-end SQL injection detection using Apache and Wazuh.

The simulated request was logged by Apache, collected by the Wazuh Agent, decoded by Wazuh, matched by Rule 31164, and displayed as a Level 6 SQL injection alert in the Wazuh Dashboard.

# Incident 01 — Windows Local User Account Creation

**Status:** Completed
**Severity:** High
**Detection Type:** Custom Wazuh Detection
**Environment:** Isolated SOC Lab
**Wazuh Version:** 4.14.7

---

## 1. Incident Summary

During this investigation, I simulated the creation of a new local user account on a Windows 7 endpoint and monitored the activity using Wazuh.

The objective was to verify that Windows Security events were being collected correctly and to create a custom Wazuh detection for local account creation.

A new Windows user account named:

```text
SOC_TestUser3
```

was created on the Windows 7 endpoint.

Windows generated **Event ID 4720**, which indicates that a user account was created.

Wazuh successfully received and analyzed the event using its Windows EventChannel decoder.

The default Wazuh rule that detected the event was:

```text
Rule ID: 60109
Description: User account enabled or created
Level: 8
```

I then created a custom detection rule:

```text
Rule ID: 100101
Level: 10
Description: Windows local user account created - SOC Lab Detection
```

The custom rule successfully detected the simulated activity.

---

# 2. Lab Environment

| Component          | Details                 |
| ------------------ | ----------------------- |
| SIEM               | Wazuh                   |
| Wazuh Version      | 4.14.7                  |
| Manager            | Ubuntu                  |
| Endpoint           | Windows 7               |
| Windows Agent      | IEWIN7                  |
| Windows IP         | 10.10.10.30             |
| Testing Machine    | Windows 7               |
| Monitoring Machine | Ubuntu                  |
| Network            | Isolated VirtualBox lab |

---

# 3. Objective

The objectives of this incident were:

1. Verify that the Windows 7 endpoint was actively connected to Wazuh.
2. Generate a controlled Windows security event.
3. Detect local account creation.
4. Identify the Windows Event ID associated with the activity.
5. Verify the default Wazuh detection.
6. Create a custom Wazuh detection rule.
7. Validate the custom rule.
8. Investigate the resulting alert.
9. Document the evidence for future SOC investigations.

---

# 4. Activity Simulation

A local Windows user account was created from the Windows 7 machine.

The command used was:

```cmd
net user SOC_TestUser3 TestPassword123! /add
```

This command was executed only inside the isolated lab environment.

The purpose was not to compromise the system, but to generate realistic Windows security telemetry that could be detected by the SIEM.

---

## Evidence 01 — Account Creation Command


```text
Windows 7
Command Prompt

net user SOC_TestUser3 hidemypassword /add

The command completed successfully.
```

---

# 5. Windows Security Event

After the account was created, Windows generated:

```text
Event ID: 4720
```

Windows Event ID 4720 represents the creation of a user account.

The event contained information about both the account that performed the action and the newly created account.

The collected event showed:

```text
Target User: SOC_TestUser
Subject User: IEUser
Event ID: 4720
Channel: Security
Computer: IEWIN7
```
![First Image](https://github.com/Jeewaka15/Wazuh-SOC-Detection-Incident-Response-Lab/blob/90abae508f6428e174190825747b9bcabfddd704/Incident_01/Evidence/(01)%20SOC_TestUser%20windows-account-creation.png)


During the final validation test, the newly generated account was:

```text
SOC_TestUser3
```

![Second Image](https://github.com/Jeewaka15/Wazuh-SOC-Detection-Incident-Response-Lab/blob/90abae508f6428e174190825747b9bcabfddd704/Incident_01/Evidence/(01.1)%20SOC_TestUser3%20windows-account-creation.png)

---



# 6. Wazuh Detection

Wazuh successfully received the Windows Security event.

The default Wazuh rule that matched the event was:

```text
Rule ID: 60109
Description: User account enabled or created
Level: 8
MITRE ATT&CK: T1098
Group:
windows
windows_security
adduser
account_changed
```

The event was associated with:

```text
Agent: IEWIN7
IP: 10.10.10.30
```

This confirmed that the Windows 7 endpoint was successfully sending Security event telemetry to Wazuh.

---

## Evidence 02 — Wazuh Event

![Third Image](https://github.com/Jeewaka15/Wazuh-SOC-Detection-Incident-Response-Lab/blob/90abae508f6428e174190825747b9bcabfddd704/Incident_01/Evidence/(02)%20wazuh-event-4720.png)


The screenshot should clearly show as many of these as possible:

* Agent: `IEWIN7`
* Event ID: `4720`
* Rule ID: `60109`
* Rule level: `8`
* Target user
* Subject user
* MITRE ATT&CK `T1098`
* Timestamp

![Fourth Image](https://github.com/Jeewaka15/Wazuh-SOC-Detection-Incident-Response-Lab/blob/90abae508f6428e174190825747b9bcabfddd704/Incident_01/Evidence/(02.1)%20Terminal-event-4720.png)

---

# 7. Initial Investigation

The event showed that the account creation activity was performed by:

```text
Subject User: IEUser
```

and created:

```text
Target User: SOC_TestUser
```

The event also contained the following account-control information:

```text
Account Disabled
Password Not Required
Normal Account
```

This is useful from a defensive perspective because account creation alone does not always mean compromise.

A SOC analyst should investigate:

* Who created the account?
* Was the account creation authorized?
* Why was the account created?
* Was a password configured?
* Was the account added to privileged groups?
* Was the account used for subsequent logons?
* Were other account properties modified?
* Did the account creation occur before other suspicious activity?

In a real environment, an unexpected local administrator account could be a persistence mechanism and would require further investigation.

---

# 8. Custom Detection Rule

To make the detection more specific to this SOC lab, I created a custom Wazuh rule based on the existing Wazuh rule `60109`.

The custom rule was:

```xml
<group name="windows,windows_security,account_creation,">

  <rule id="100101" level="10">
    <if_sid>60109</if_sid>
    <description>Windows local user account created - SOC Lab Detection</description>
    <mitre>
      <id>T1098</id>
    </mitre>
    <group>account_creation,windows_security,</group>
  </rule>

</group>
```

---

# 9. Why a Custom Rule?

The existing Wazuh rule already detected the activity.

However, the purpose of this exercise was to practice detection engineering rather than relying only on default detections.

The custom rule:

* Uses `60109` as the parent detection.
* Assigns a higher severity of `10`.
* Provides a lab-specific description.
* Adds an `account_creation` classification.
* Maintains the MITRE ATT&CK mapping.

This approach can be useful when developing organization-specific detections on top of existing SIEM rules.

---

# 10. Rule Validation

Before restarting the Wazuh Manager, the configuration was tested using:

```bash
sudo /var/ossec/bin/wazuh-analysisd -t
```

The Wazuh Manager was then restarted:

```bash
sudo systemctl restart wazuh-manager
```

The manager status was verified to ensure that the service was running correctly.

---

## Evidence 03 — Wazuh Rule Configuration

![Fifth Image](https://github.com/Jeewaka15/Wazuh-SOC-Detection-Incident-Response-Lab/blob/90abae508f6428e174190825747b9bcabfddd704/Incident_01/Evidence/(03)%20custom-rule-100101.png)

The screenshot should show:

```text
<rule id="100101" level="10">
```

and:

```text
Windows local user account created - SOC Lab Detection
```

---

# 11. Custom Detection Test

After the Wazuh Manager was restarted, another controlled account creation event was generated on Windows 7:

```cmd
net user SOC_TestUser3 hidemypassword /add
```

The Wazuh alert data was then searched for the custom rule:

```bash
sudo grep -a '"id":"100101"' /var/ossec/logs/alerts/alerts.json | tail -1
```

The search successfully returned the custom rule.

This confirmed that the custom rule was not only configured correctly but also successfully triggered against the Windows event.

---

## Evidence 04 — Custom Rule Alert

![Sixth Image](https://github.com/Jeewaka15/Wazuh-SOC-Detection-Incident-Response-Lab/blob/90abae508f6428e174190825747b9bcabfddd704/Incident_01/Evidence/(04)%20custom-alert-terminal.png)

The important fields to capture are:

```text
rule.id: 100101
rule.level: 10
description: Windows local user account created - SOC Lab Detection
agent.name: IEWIN7
agent.ip: 10.10.10.30
```

---

# 12. Wazuh Dashboard Verification

The event was also verified through the Wazuh Dashboard.

The relevant detection can be searched using:

```text
rule.id:100101
```

The expected detection contains:

```text
Rule ID: 100101
Level: 10
Agent: IEWIN7
```

---

## Evidence 05 — Wazuh Dashboard Custom Detection

![Seventh Image](https://github.com/Jeewaka15/Wazuh-SOC-Detection-Incident-Response-Lab/blob/90abae508f6428e174190825747b9bcabfddd704/Incident_01/Evidence/(05)%20wazuh-dashboard-rule-100101.png)

This is one of the **most important screenshots** for the GitHub report.

Try to capture the event details panel where possible.

![Final Image](https://github.com/Jeewaka15/Wazuh-SOC-Detection-Incident-Response-Lab/blob/90abae508f6428e174190825747b9bcabfddd704/Incident_01/Evidence/(05.1)%20wazuh-dashboard-rule-100101.png)

---

# 13. MITRE ATT&CK Mapping

The detection is mapped to:

### T1098 — Account Manipulation

Account manipulation covers techniques where adversaries modify accounts or account properties to maintain access or increase their capabilities.

In this lab, the relevant activity is the creation of a new local Windows account.

```text
MITRE ATT&CK Technique: T1098
Tactic: Persistence
```

The mapping is based on the Wazuh rule that identified the account-related activity.

---

# 14. Timeline

| Time    | Activity                                                 |
| ------- | -------------------------------------------------------- |
| Initial | Windows 7 endpoint connected to Wazuh                    |
| T1      | User account creation activity generated                 |
| T2      | Windows generated Event ID 4720                          |
| T3      | Wazuh received the Windows Security event                |
| T4      | Rule 60109 detected the activity                         |
| T5      | Custom rule 100101 was configured                        |
| T6      | Wazuh Manager configuration was validated                |
| T7      | Wazuh Manager restarted                                  |
| T8      | Account creation activity was generated again            |
| T9      | Rule 100101 successfully triggered                       |
| T10     | Custom detection verified through Wazuh alerts/dashboard |

---

# 15. Evidence Collected

The following evidence was collected during the investigation:

### Evidence A — Windows Activity

```text
net user SOC_TestUser3 hidemypassword /add
```

Purpose:

Generate a controlled local account creation event.

---

### Evidence B — Windows Event

```text
Event ID: 4720
Channel: Security
Computer: IEWIN7
```

Purpose:

Confirm that Windows recorded the account creation.

---

### Evidence C — Default Wazuh Detection

```text
Rule ID: 60109
Level: 8
Description: User account enabled or created
```

Purpose:

Confirm that Wazuh successfully detected the Windows event.

---

### Evidence D — Custom Detection

```text
Rule ID: 100101
Level: 10
Description: Windows local user account created - SOC Lab Detection
```

Purpose:

Confirm successful custom detection engineering.

---

### Evidence E — Agent Information

```text
Agent: IEWIN7
IP: 10.10.10.30
```

Purpose:

Identify the endpoint that generated the event.

---

# 16. Analyst Assessment

The simulated activity was successfully detected.

The investigation demonstrated the complete flow from endpoint activity to SIEM detection:

```text
Windows 7
   ↓
Local account creation
   ↓
Windows Event ID 4720
   ↓
Wazuh Agent
   ↓
Wazuh Manager
   ↓
Rule 60109
   ↓
Custom Rule 100101
   ↓
Level 10 Alert
   ↓
SOC Investigation
```

The detection pipeline is functioning correctly.

In a real enterprise environment, an unexpected account creation event should not automatically be classified as malicious. The analyst should correlate the event with authentication logs, account privileges, process activity, administrator actions, and change-management records.

---

# 17. Detection Improvement Opportunities

This detection can be improved further in future versions of the lab.

Possible improvements include:

* Detecting accounts added to local Administrators.
* Detecting accounts with suspicious account-control settings.
* Detecting account creation followed by successful login.
* Detecting account creation outside normal administrative activity.
* Correlating account creation with suspicious processes.
* Detecting account creation followed by remote access.
* Creating higher-severity rules for privileged account creation.
* Adding alert enrichment and automated response.

---

# 18. Lessons Learned

This incident helped me understand several practical SOC concepts.

### 1. Log collection is the foundation of detection

The Windows endpoint first needed to generate and forward the security event before Wazuh could detect it.

### 2. Default rules are useful starting points

Wazuh's existing rule `60109` successfully identified the account creation event.

### 3. Custom rules provide more control

The custom rule allowed the detection to be classified specifically for this SOC lab.

### 4. Detection requires investigation

An alert does not automatically mean that an attack occurred.

The analyst needs to understand the context around the event.

### 5. Evidence is important

The Windows event, Wazuh alert, rule configuration, and command output provide evidence that the detection actually worked.

---

# 19. Final Result

**Detection Status: SUCCESS**

The lab successfully demonstrated:

```text
✓ Windows 7 endpoint monitoring
✓ Windows Security Event collection
✓ Event ID 4720 detection
✓ Default Wazuh Rule 60109
✓ Custom Wazuh Rule 100101
✓ Level 10 custom alert
✓ MITRE ATT&CK mapping
✓ Threat investigation
✓ Evidence collection
✓ SOC incident documentation
```

This incident is now being used as the first documented case in the Wazuh SOC Lab.

---

## 20. Files / Configuration Used

### Wazuh Custom Rule

```text
/var/ossec/etc/rules/local_rules.xml
```

### Wazuh Alert Log

```text
/var/ossec/logs/alerts/alerts.json
```

### Configuration Test

```bash
sudo /var/ossec/bin/wazuh-analysisd -t
```

### Wazuh Manager Restart

```bash
sudo systemctl restart wazuh-manager
```

### Custom Alert Verification

```bash
sudo grep -a '"id":"100101"' /var/ossec/logs/alerts/alerts.json | tail -1
```

### Windows Account Creation

```cmd
net user SOC_TestUser3 hidemypassword /add
```

---



## 22. Conclusion

Incident 01 successfully demonstrated a complete SOC detection workflow using Wazuh.

A controlled Windows local account creation event was generated, collected from the Windows endpoint, detected by Wazuh, investigated using the available event information, mapped to MITRE ATT&CK, and finally detected using a custom Wazuh rule.

The next stage of the project will focus on **network reconnaissance detection using Kali Linux against the Windows 7 lab environment**, followed by investigation and documentation using the same SOC workflow.

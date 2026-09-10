# Wazuh SOC Lab 🛡️⚡🔬  Detection & Response  🚨🎯💥

## Overview

This repository documents my hands-on Security Operations Center (SOC) lab built using **Wazuh, Windows 7, Ubuntu, and Kali Linux**.

The main purpose of this project is to move beyond theoretical cybersecurity knowledge and gain practical experience in security monitoring, log analysis, threat detection, threat hunting, and incident investigation.

Instead of only generating alerts, I am following a simple SOC workflow:

**Simulate Activity → Generate Logs → Collect Logs → Detect → Investigate → Map to MITRE ATT&CK → Document Evidence → Improve Detection**

The project is currently **in progress**, and new incidents, detection rules, and investigations will be added as I continue developing the lab.

---

## Lab Environment

| Component     | Purpose                                                        |
| ------------- | -------------------------------------------------------------- |
| Wazuh         | SIEM / XDR platform for log collection, analysis and detection |
| Ubuntu        | Wazuh Manager / SOC monitoring machine                         |
| Windows 7     | Monitored endpoint used for security event generation          |
| Kali Linux    | Security testing and reconnaissance machine                    |
| VirtualBox    | Virtual lab environment                                        |
| Wazuh Version | 4.14.7                                                         |
| Windows 7 IP  | 10.10.10.30                                                    |

The machines are connected through an isolated virtual lab network.

---

## What I Am Doing

The project is being developed as a small SOC environment where I can simulate realistic security activities and investigate how they appear from a defender's perspective.

### 1. Endpoint Monitoring

I configured the Windows 7 machine as a Wazuh agent and connected it to the Wazuh Manager.

The objective is to monitor activities such as:

* User account creation
* User account changes
* Authentication events
* Service changes
* System events
* Process and security-related activity
* Other Windows Security and System events

---

### 2. Attack / Security Activity Simulation

The lab will use controlled activities to generate security telemetry.

Examples include:

* Creating local Windows accounts
* Changing account properties
* Network reconnaissance using Kali Linux
* Scanning exposed services
* Authentication-related activities
* Service modifications
* Suspicious system activity

All activities are performed inside the isolated lab environment for learning and defensive security testing.

---

### 3. Detection Engineering

I am also creating custom Wazuh rules when the default rules are not sufficient for the type of detection I want to demonstrate.

For example, during **Incident 01**, Wazuh detected Windows Event ID **4720**, which represents a user account being created.

The default Wazuh rule was:

```text
Rule ID: 60109
Description: User account enabled or created
Level: 8
```

I created an additional custom rule:

```text
Rule ID: 100101
Level: 10
Description: Windows local user account created - SOC Lab Detection
```

This allows the activity to be highlighted as a higher-priority SOC detection.

---

## 4. Threat Hunting

After an event is generated, I search the Wazuh dashboard and alert data to understand:

* What happened?
* Which endpoint generated the event?
* Which user performed the action?
* What changed?
* When did it happen?
* Is the activity expected or suspicious?
* What MITRE ATT&CK technique is relevant?
* What additional evidence should be investigated?

The goal is to develop an investigation mindset rather than simply looking at alert severity.

---

## 5. Incident Investigation

Each important security event will be documented as an individual incident.

Each incident will contain:

* Incident summary
* Lab environment
* Attack / activity simulation
* Detection details
* Wazuh rule information
* Windows Event ID
* MITRE ATT&CK mapping
* Timeline
* Investigation
* Evidence
* Detection logic
* Analyst conclusion
* Recommended improvements
* Screenshots and command outputs

This makes the repository a record of the SOC investigations performed during the project.

---

## Incident Tracking

| Incident    | Activity                            | Detection                   | Status      |
| ----------- | ----------------------------------- | --------------------------- | ----------- |
| Incident 01 | Windows local user account creation | Event ID 4720 / Rule 100101 | Completed   |
| Incident 02 | Network reconnaissance              | Planned                     | In Progress |
| Incident 03 | Authentication activity             | Planned                     | Planned     |
| Incident 04 | Service modification                | Planned                     | Planned     |
| Incident 05 | Suspicious process activity         | Planned                     | Planned     |

This table will be updated as the project progresses.

---

## Project Goals

The main goals of this project are to gain practical experience with:

* SIEM monitoring
* Wazuh
* Windows event analysis
* Linux administration
* Security log analysis
* Detection engineering
* Custom Wazuh rules
* Threat hunting
* MITRE ATT&CK mapping
* Network reconnaissance detection
* Incident investigation
* Evidence collection
* SOC documentation

---

## Current Progress

### Completed

* Wazuh Manager configured
* Windows 7 Wazuh agent connected
* Windows event collection verified
* Windows Security events received by Wazuh
* Windows Event ID 4720 successfully detected
* Custom Wazuh rule `100101` created
* Custom rule tested successfully
* Incident 01 documented

### In Progress

* Kali Linux reconnaissance detection
* Network-based threat hunting
* Additional custom detection rules
* Incident response documentation
* Additional attack simulations

---

## Project Philosophy

This is not intended to be a production SOC environment.

It is a controlled learning lab designed to understand how attacks and suspicious activities generate telemetry and how a SOC analyst can use that telemetry to detect, investigate, and document security incidents.

The project will continue to evolve as I learn new security concepts and techniques.

---

## Repository Structure

```text
Wazuh-SOC-Lab/
│
├── Overview.md
│
├── Incident_01/
│   └── incident_01.md
│
├── Incident_02/
│   └── incident_02.md
│
├── Incident_03/
│   └── incident_03.md
│
└── README.md
```

Each incident folder contains the investigation report and supporting evidence related to that incident.

---

## Disclaimer

All security testing and attack simulations in this project are performed inside my own isolated virtual laboratory.

The techniques demonstrated in this repository should only be used on systems where you have explicit authorization to perform security testing.

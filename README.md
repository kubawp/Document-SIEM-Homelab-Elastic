# Document-SIEM-Homelab-Elastic
Security monitoring and threat detection lab using Elasticsearch, Kibana, ElastAlert and Beats.

## Project Goals

The main goal of this project was to build a functional security monitoring environment that simulates a small SOC-like infrastructure.

The project includes:

- Centralized log collection from Linux and Windows hosts
- Deployment of Elasticsearch and Kibana as the SIEM backend
- Integration of multiple Beats agents
- Custom ElastAlert detection rules
- Kibana dashboards for authentication, network and system monitoring

## Infrastructure Overview

The environment consists of three main parts:

- **SIEM Server** — Ubuntu server hosting Elasticsearch, Kibana and ElastAlert
- **Linux Host** — Kali Linux machine generating Linux logs and network telemetry
- **Windows Host** — Windows 11 machine generating Windows event logs

The central SIEM server was deployed on an Ubuntu virtual machine hosted in Microsoft Azure. To reduce the attack surface and limit unauthorized access, administrative services were protected using Azure NSG rules. Access to management interfaces was restricted to trusted source address.

Logs and telemetry are collected from endpoint machines and sent to the central Elasticsearch instance. Kibana is used for visualization, while ElastAlert is responsible for rule-based detection and alert generation.

![SIEM Homelab Infrastructure](./images/siem_infra.png)

## Technology Stack

| Component | Purpose |
|-----------|---------|
| Elasticsearch | Centralized log storage and search engine |
| Kibana | Log analysis and visualization |
| ElastAlert | Detection rules and alerting |
| Filebeat | Linux log collection |
| Winlogbeat | Windows Event Log collection |
| Packetbeat | Network traffic monitoring |
| Auditbeat | File and system activity monitoring |
| Metricbeat | System performance monitoring |

# Attack Simulation & Detection

The following scenarios were implemented to validate the effectiveness of the deployed SIEM environment.

# SSH Brute Force Detection (Linux)

A brute-force attack against the Linux SSH service was simulated using Hydra. 

## Step 1 – Attack Simulation

![SSH Brute Force Simulation - Hydra](./images/ssh_brute_force_hydra.png)

---

## Step 2 – Event Collection

The authentication attempts generated during the attack were collected from Linux authentication logs by Filebeat and forwarded to Elasticsearch.

![SSH Brute Force Simulation - Logs](./images/ssh_brute_force_logs.png)

---

## Step 3 – Detection

The custom ElastAlert rule detected multiple failed authentication attempts.

Rule:

➡️ `detection-rules/linux/ssh_brute_force.yaml`

> **Note:** The detection threshold used in this laboratory environment was intentionally lowered to enable quick validation during testing. In a production environment, thresholds and time windows should be adjusted to reflect normal user behavior and reduce the likelihood of false positives.

![SSH Brute Force Simulation - Alert](./images/ssh_brute_force_alert.png)

---

# Successful SSH Brute Force Attack Detection (Linux)

Following the brute-force attack, Hydra successfully authenticated using the discovered password. To emulate a realistic attack scenario, the target account password was intentionally changed to a commonly used password contained in the `rockyou.txt` wordlist. This allowed Hydra to successfully discover valid credentials.

A correlation rule was implemented to detect successful authentication following a previously detected brute-force attack.

## Step 1 – Attack Simulation and Successful Authentication

![SSH Successful Brute Force Simulation - Hydra](./images/ssh_successful_brute_force_hydra.png)

---

## Step 2 – Event Collection

Filebeat collected both the failed authentication attempts and the final successful login event.

![SSH Successful Brute Force Simulation - Logs](./images/ssh_successful_brute_force_logs.png)

---

## Step 3 – Detection

The correlation rule identified a successful login occurring shortly after a brute-force attack.

Rule:

➡️ `detection-rules/linux/ssh_succesful_brute_force.yaml`

![SSH Successful Brute Force Simulation - Alert](./images/ssh_successful_brute_force_alert.png)

---

# Successful RDP Login after Brute Force (Windows)

Following multiple failed authentication attempts, a successful RDP login was performed.

A correlation rule was created to detect a successful login immediately following a previously detected brute-force attack.

## Step 1 – Event Collection

Winlogbeat collected both failed and successful Windows authentication events (Event ID's 4625 and 4624).

![RDP Successful Brute Force Simulation - Logs](./images/rdp_successful_brute_force_logs.png)

---

## Step 2 – Detection

The correlation rule generated an alert indicating a successful login after a brute-force attack.

Rule:

➡️ `detection-rules/windows/windows_rdp_successful_brute_force.yaml`

![RDP Successful Brute Force Simulation - Alert](./images/rdp_successful_brute_force_alert.png)

---

# Kibana Dashboards

The project includes several dashboards supporting security monitoring and incident investigation.

## Linux Authentication Dashboard

This dashboard provides visibility into Linux authentication activity collected by Filebeat.

It includes:

- SSH successful and failed login attempts
- FTP successful and failed login attempts
- Recently executed sudo commands
- Distribution of Linux system processes

![Linux Authentication Dashboard](./images/filebeat_dashboard.png)

---

## Windows Authentication Dashboard

This dashboard focuses on Windows authentication events collected by Winlogbeat.

It provides:

- Successful and failed SSH logins
- Successful and failed RDP logins
- Windows Event Log severity overview
- Security event summary

![Windows Authentication Dashboard](./images/winlogbeat_dashboard.png)

---



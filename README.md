# Honeypot Lab: Cloud SOC with Microsoft Sentinel

A cloud-based Security Operations Centre (SOC) built in Azure. A deliberately exposed honeypot VM captures real-world RDP and SSH brute-force attacks. Logs flow into Microsoft Sentinel, where attacks are investigated with KQL, visualised on geographic heat maps, and escalated by automated alerts. The infrastructure is then scanned with Tenable Nessus and hardened to security best practice.


## Objectives

1. Deploy a honeypot VM and collect real attack telemetry.
2. Centralise logs in a Log Analytics workspace and Microsoft Sentinel.
3. Detect and analyse brute-force activity with KQL.
4. Visualise attack origin on a Sentinel Workbook map.
5. Automate alerting with an analytics rule and a Logic App playbook.
6. Run Nessus vulnerability scans, remediate findings, and re-scan to prove improvement.

## Architecture

```
Internet attackers
      |  RDP 3389 / SSH 22
      v
+---------------------+       +-----------------------------+
| Azure NSG (open)    |       | Log Analytics Workspace     |
| Honeypot VM(s)      | ----> |  + Microsoft Sentinel       |
| Win (RDP) / Linux   |  AMA  |  - SecurityEvent / Syslog   |
+---------------------+       |  - Analytics rules (KQL)    |
                              |  - Workbook (heat map)      |
                              +--------------+--------------+
                                             | Incident
                                             v
                              +-----------------------------+
                              | Logic App playbook          |
                              |  -> Email / Teams alert     |
                              +-----------------------------+

Nessus scanner  --credentialed scan-->  Honeypot VM / management VM
```

See [docs/01-architecture.md](docs/01-architecture.md) for detail.

## Tech stack

| Area | Tool |
|---|---|
| Cloud | Microsoft Azure (VMs, NSG, Resource Groups) |
| SIEM / SOAR | Microsoft Sentinel, Log Analytics, Logic Apps |
| Query language | KQL |
| Vulnerability management | Tenable Nessus |
| Honeypot OS | Windows Server / Windows 10 (RDP), Ubuntu (SSH) |

## Repository layout

```
honeypot-lab/
├── README.md
├── docs/
│   ├── 01-architecture.md
│   ├── 02-deployment-guide.md
│   ├── 03-kql-analysis.md
│   ├── 04-alerting-and-automation.md
│   ├── 05-vulnerability-management.md
│   ├── 06-findings-report-template.md
│   ├── 07-security-and-cost-guardrails.md
│   └── 08-interview-prep.md
├── queries/            # ready-to-run .kql files
├── logic-app/          # playbook definition notes + email template
└── evidence/           # put your screenshots here
```

## Quick start

1. Read [docs/07-security-and-cost-guardrails.md](docs/07-security-and-cost-guardrails.md) first.
2. Follow [docs/02-deployment-guide.md](docs/02-deployment-guide.md).
3. Let the honeypot run 24-72 hours before analysing.
4. Work through the docs in order, saving screenshots to `evidence/`.
5. Complete [docs/06-findings-report-template.md](docs/06-findings-report-template.md) with your real data.
6. Rehearse with [docs/08-interview-prep.md](docs/08-interview-prep.md).

## Skills demonstrated

Cloud security, SIEM operations, threat detection engineering, KQL, log analysis, SOAR automation, vulnerability management, hardening and remediation, and security documentation.

## Disclaimer

This is an educational lab. Run it only in your own Azure subscription, isolated from any real data or networks.

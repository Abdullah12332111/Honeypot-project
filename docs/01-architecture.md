# 01 - Architecture

## Goal

Capture real attacker behaviour against an intentionally exposed system, then practise the full SOC loop: **collect, detect, investigate, respond, harden**.

## Components

| Component | Purpose |
|---|---|
| Resource group `rg-honeypot-lab` | Single container so everything can be deleted in one step |
| Virtual network + subnet | Isolated network, no peering to anything real |
| Honeypot VM (Windows) | Exposes RDP (3389). Generates Event ID 4625 failed logons |
| Honeypot VM (Ubuntu), optional | Exposes SSH (22). Generates auth failures in Syslog |
| Network Security Group (NSG) | Deliberately allows inbound 3389/22 from `Any` on the honeypot subnet |
| Log Analytics workspace `law-honeypot` | Log store and query engine |
| Microsoft Sentinel | SIEM layer on the workspace: analytics rules, incidents, workbooks |
| Azure Monitor Agent (AMA) + Data Collection Rule | Ships Windows Security events and Linux Syslog to the workspace |
| Logic App playbook | SOAR automation triggered by Sentinel incidents |
| Management VM with Nessus | Scanner used in the vulnerability management phase |

## Data flow

1. Internet scanners find the open RDP/SSH port within hours.
2. Attackers attempt logons. Windows logs Event ID **4625** (failed) and **4624** (success). Linux logs `Failed password` / `Invalid user` to auth log.
3. AMA forwards events via a Data Collection Rule to Log Analytics.
4. Sentinel analytics rules run KQL on a schedule and raise incidents.
5. An automation rule runs a Logic App playbook that notifies the analyst.
6. Analyst investigates in Logs and Workbooks, enriches with geolocation.

## Why the honeypot has no real data

The VM is treated as hostile and disposable. It holds no credentials, no domain join, no VPN links and no real data. An attacker who compromised it would gain nothing. See [07-security-and-cost-guardrails.md](07-security-and-cost-guardrails.md).

## Geolocation approach

Failed-logon events contain only an IP address. Two common ways to map them:

- **Sentinel Watchlist** containing a GeoIP CSV, joined in KQL with `ipv4_lookup`.
- **`geo_info_from_ip_address()`** KQL function, which needs no external file.

This project uses the built-in function (simpler, nothing to maintain). Both approaches are documented in [03-kql-analysis.md](03-kql-analysis.md).

## Diagram (add your own)

Draw a diagram in draw.io or Excalidraw that matches your real deployment and save it as `evidence/architecture.png`. Interviewers like candidates who can draw their own design on a whiteboard.

# 02 - Deployment Guide

Read [07-security-and-cost-guardrails.md](07-security-and-cost-guardrails.md) before you start. Portal menu names change occasionally; if a label differs, search for the feature name.

## Prerequisites

- An Azure subscription (free trial credit is enough).
- A budget alert set (see guardrails doc).
- Permission to create resources and enable Sentinel (Owner or Contributor on the subscription/resource group).

## Step 1 - Resource group and region

1. Create resource group `rg-honeypot-lab` in a region close to you (e.g. UK South).
2. Tag it `purpose=honeypot` and `delete-after=<date>`.

## Step 2 - Log Analytics workspace and Sentinel

1. Create Log Analytics workspace `law-honeypot` in the same resource group.
2. In the portal open **Microsoft Sentinel**, choose **Create**, and select `law-honeypot`.
3. Note the retention period (default 30 days is fine for the lab).

## Step 3 - Network

1. Create VNet `vnet-honeypot` (e.g. 10.10.0.0/16) with subnet `snet-honeypot` (10.10.1.0/24).
2. Create a second subnet `snet-mgmt` (10.10.2.0/24) for the Nessus scanner.

## Step 4 - Honeypot VM (Windows, RDP)

1. Create a Windows Server VM `vm-honeypot-win` in `snet-honeypot`. Small size (e.g. B2s) is enough.
2. Use a strong, unique local admin password. The honeypot should capture failed attempts, not be trivially compromised.
3. Create an NSG for the NIC. Add inbound rule: **Destination port 3389, Source Any, Allow, priority 100**. Delete or deprioritise the default-deny override so the rule takes effect.
4. Optional: turn off Windows Firewall inside the VM so probes are not blocked at the OS level. Do this only on this isolated VM.
5. Do not join a domain. Do not store anything real on it.

## Step 5 - Honeypot VM (Linux, SSH), optional

1. Create Ubuntu VM `vm-honeypot-lin` in `snet-honeypot`.
2. Use SSH key auth for admin access via a different port or Azure Bastion if you want to manage it safely, while leaving port 22 exposed as the bait. Simplest safe option: keep password auth enabled on 22 with a long random password.
3. NSG rule: **port 22, Source Any, Allow**.

## Step 6 - Collect logs

1. In Sentinel go to **Content hub** and install **Windows Security Events** and **Syslog**.
2. Open the **Windows Security Events via AMA** connector, create a Data Collection Rule scoped to `vm-honeypot-win`, and choose **All Security Events** (or Custom with Event IDs 4624, 4625, 4688, 4672).
3. Open the **Syslog via AMA** connector, create a DCR for `vm-honeypot-lin`, facility `auth` and `authpriv`, level `Info`.
4. Confirm the Azure Monitor Agent extension installed on each VM.

## Step 7 - Verify data arrives

Wait 15 to 30 minutes, then in **Logs** run:

```kql
SecurityEvent
| where TimeGenerated > ago(1h)
| summarize count() by EventID
```

and for Linux:

```kql
Syslog
| where TimeGenerated > ago(1h)
| summarize count() by Facility, SeverityLevel
```

Take a screenshot of the output as evidence.

## Step 8 - Let it run

Leave the honeypot exposed for **24 to 72 hours**. Most labs see the first brute-force attempts within the first hour. Check **Logs** daily.

## Step 9 - Continue

- Analysis: [03-kql-analysis.md](03-kql-analysis.md)
- Alerting: [04-alerting-and-automation.md](04-alerting-and-automation.md)
- Scanning and hardening: [05-vulnerability-management.md](05-vulnerability-management.md)

## Teardown

When finished, delete the resource group. This removes every resource and stops billing.

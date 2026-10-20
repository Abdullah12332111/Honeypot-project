# 04 - Alerting and Automation

## Part A - Analytics rule (detection)

1. Sentinel > **Analytics** > **Create > Scheduled query rule**.
2. Name: `Honeypot - RDP brute force from single IP`.
3. Severity: Medium. MITRE ATT&CK: **Credential Access > T1110 Brute Force** (sub-technique T1110.001 Password Guessing, T1110.003 Password Spraying).
4. Rule query:

```kql
SecurityEvent
| where EventID == 4625
| where isnotempty(IpAddress) and IpAddress != "-"
| summarize FailedAttempts = count(), Accounts = dcount(TargetUserName),
            FirstSeen = min(TimeGenerated), LastSeen = max(TimeGenerated)
            by IpAddress, Computer
| where FailedAttempts >= 20
| extend IPCustomEntity = IpAddress, HostCustomEntity = Computer
```

5. Run every **5 minutes**, look back **15 minutes**. Alert threshold: greater than 0 results.
6. **Entity mapping**: IP > `IPCustomEntity`, Host > `HostCustomEntity`.
7. Incident settings: create incidents, group alerts by IP within 5 hours.

A second rule for the dangerous case (success after failures) uses query 7 from [03-kql-analysis.md](03-kql-analysis.md). Set that one to **High** severity, MITRE **Initial Access / T1078 Valid Accounts**.

Add a Linux rule using the SSH query in the same way.

### Tuning

Honeypots are noisy by design. Set the threshold so the lab generates a manageable number of incidents. Document your threshold and why you chose it. Tuning rules to cut noise is a real SOC skill and a good interview topic.

## Part B - Logic App playbook (response)

Goal: when an incident is created, email (or Teams-message) the analyst with key details.

1. Create a Logic App (Consumption) named `la-honeypot-notify` in `rg-honeypot-lab`.
2. Trigger: **Microsoft Sentinel incident** (when Microsoft Sentinel incident creation rule was triggered).
3. Actions, in order:
   1. **Initialize variable** `IncidentTitle` from the trigger.
   2. **Entities - Get IPs** (Sentinel action) to extract the attacker IP.
   3. **For each** IP, **HTTP** GET to a free IP reputation/geolocation API of your choice (optional enrichment; check the API's terms and rate limits).
   4. **Send an email (V2)** using Office 365 Outlook or Gmail connector, using the template in [`/logic-app/email-template.html`](../logic-app/email-template.html).
   5. **Add comment to incident** noting that the notification was sent.
4. Grant the Logic App's managed identity the **Microsoft Sentinel Responder** role on the workspace.
5. In Sentinel > **Automation** create an **automation rule**: when incident is created, run playbook `la-honeypot-notify`.

## Test it

Generate a test: from your own machine, attempt several wrong RDP passwords against the honeypot (your own lab only), or simply wait for real traffic. Confirm the incident appears, the email arrives, and the incident gets the comment. Screenshot each step.

## Optional extensions

- Auto-add attacker IPs to a Sentinel Watchlist (`blocked-ips`).
- Playbook step to add an NSG deny rule for repeat offenders (use carefully; keep the honeypot exposed to new sources so you keep collecting data).
- Post to a Teams channel instead of email.

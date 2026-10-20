# 06 - Findings Report (Template)

Fill this in with your own data. Replace every `[ ]` placeholder. Do not invent figures.

---

# Honeypot Lab: Findings Report

**Author:** [Your name]
**Date range of data collection:** [start] to [end]
**Environment:** Azure, Microsoft Sentinel, Log Analytics, Logic Apps, Nessus

## 1. Executive summary

[3 to 5 sentences. What you built, how long it ran, the headline numbers, and the main security lessons.]

Example structure: "Over [N] hours, the honeypot recorded [N] failed logon attempts from [N] unique IP addresses across [N] countries. The first attempt arrived [N] minutes after exposure. Nessus identified [N] Critical/High findings, all remediated and verified by re-scan."

## 2. Methodology

- Honeypot hosts: [OS, size, exposed ports]
- Log collection: [AMA, DCR, event IDs]
- Detection: [analytics rules and thresholds]
- Response: [Logic App behaviour]
- Vulnerability scanning: [Nessus version, scan types]

## 3. Attack statistics

| Metric | Value |
|---|---|
| Collection period | [ ] |
| Total failed logons (RDP) | [ ] |
| Total failed logons (SSH) | [ ] |
| Unique source IPs | [ ] |
| Unique countries | [ ] |
| Time to first attack after exposure | [ ] |
| Peak hour (UTC) | [ ] |
| Successful unauthorised logons | [ ] |

### Top 5 source countries

| Rank | Country | Attempts |
|---|---|---|
| 1 | [ ] | [ ] |
| 2 | [ ] | [ ] |
| 3 | [ ] | [ ] |
| 4 | [ ] | [ ] |
| 5 | [ ] | [ ] |

### Top 10 targeted usernames

[Table or list from query 3]

![Attack map](../evidence/attack-map.png)

## 4. Attack pattern analysis

[Brute force or spray? Automated or slow-and-low? Common usernames? Any IPs that appeared repeatedly over days? Any reuse across RDP and SSH? Mapped to MITRE ATT&CK T1110.]

## 5. Detection and response

- Rules created: [names, severity, thresholds]
- Number of incidents generated: [ ]
- Tuning done: [what you changed and why]
- Playbook result: [screenshot of email and incident comment]
- Mean time from alert to notification: [ ]

## 6. Vulnerability assessment

### Before vs after

| Severity | Before | After |
|---|---|---|
| Critical | [ ] | [ ] |
| High | [ ] | [ ] |
| Medium | [ ] | [ ] |
| Low | [ ] | [ ] |

### Notable findings and fixes

| Finding | Plugin / CVE | Severity | Fix applied | Verified |
|---|---|---|---|---|
| [ ] | [ ] | [ ] | [ ] | [ ] |

## 7. Recommendations

1. Never expose RDP or SSH directly to the internet; use Bastion, VPN or JIT access.
2. Enforce account lockout and strong passwords; prefer key-based SSH and MFA.
3. Keep systems patched; track with regular authenticated scans.
4. Centralise logs and alert on repeated failures and success-after-failure.
5. [Add your own, based on your results]

## 8. Lessons learned and limitations

- [What you would do differently]
- [Limits: single region, short window, one honeypot, no real data at risk]

## Appendix

- KQL queries: `/queries`
- Screenshots: `/evidence`

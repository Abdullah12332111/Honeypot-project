# 08 - Interview Prep

## 30-second pitch

"I built a cloud SOC in Azure. I exposed a honeypot VM to the internet and captured real RDP and SSH brute-force attacks. I shipped the logs into Microsoft Sentinel, analysed attack patterns with KQL, mapped the sources on a heat map, and automated alerting with Logic Apps. I then scanned the environment with Nessus, fixed the findings, and re-scanned to show the improvement."

## 2-minute walkthrough (STAR)

- **Situation:** I wanted hands-on SOC experience, not just theory.
- **Task:** Build an environment that produces real attack data and run the full detect-investigate-respond-harden cycle.
- **Action:** Deployed the honeypot and Sentinel, wrote KQL detections, built a workbook map, built a Logic App playbook, ran Nessus before and after remediation.
- **Result:** [Your real numbers: attempts, unique IPs, countries, time to first attack, findings fixed.]

## Likely questions and strong answers

**Why a honeypot?**
It produces real, current attacker behaviour instead of simulated data, and it is safe if isolated, so I could practise detection on realistic traffic.

**How did you collect the logs?**
Azure Monitor Agent with a Data Collection Rule sending Windows Security events (4624, 4625) and Linux Syslog auth logs to a Log Analytics workspace behind Sentinel.

**Which Windows event IDs matter here?**
4625 failed logon, 4624 successful logon, 4672 privileged logon, 4688 process creation. Logon type 10 is RDP.

**How do you tell brute force from password spraying?**
Brute force is many guesses against one account. Spraying is few guesses against many accounts. In KQL I compare attempts and distinct usernames per source IP.

**How did you map attackers geographically?**
`geo_info_from_ip_address()` in KQL (or a GeoIP watchlist with `ipv4_lookup`), then a Sentinel Workbook with a Map visualisation using latitude/longitude and attempts for size.

**What detection rules did you write and how did you tune them?**
A scheduled rule alerting at [N] failed logons from one IP in 15 minutes, plus a high-severity rule for a successful logon after repeated failures. I set thresholds to balance noise against missing real attacks. [Describe your tuning.]

**What does your Logic App do?**
It triggers from a Sentinel incident, extracts the attacker IP entity, sends an email notification, and comments on the incident. Access uses a managed identity with the Sentinel Responder role.

**What is MITRE ATT&CK mapping for this?**
T1110 Brute Force (T1110.001 password guessing, T1110.003 spraying), and T1078 Valid Accounts for a successful login.

**What did Nessus find and what did you fix?**
[Your real findings.] Common categories are missing patches, RDP without NLA, weak TLS, SMBv1, and weak password policy. I re-scanned to verify the fixes and compared before and after counts.

**How do you prioritise vulnerabilities?**
By severity (CVSS), exploitability, exposure, asset value, and ease of fix. Internet-facing and actively exploitable first.

**What would you do if the attacker had succeeded?**
Isolate the VM (NSG deny or deallocate), preserve evidence (snapshot disk, export logs), identify the account and time of access, review 4688 process events for post-compromise activity, rotate any exposed credentials, and rebuild from a clean image.

**How would you secure RDP/SSH in production?**
No direct internet exposure. Use Bastion, VPN or Just-in-Time access, MFA, NLA, account lockout, key-based SSH, patching, and monitoring.

**What were the limitations?**
Single region, short window, one or two honeypot hosts, and no insight into attackers beyond logs. The data shows opportunistic scanning, not targeted attacks.

**What would you add next?**
Threat intelligence enrichment, auto-blocking repeat offenders, more honeypot services, Defender for Cloud integration, and infrastructure-as-code (Bicep/Terraform) for repeatable deployment.

## Be ready to demo

Have these open: the workbook map, the failed logon timechart, an incident with its entities, the Logic App run history, and the Nessus before/after summaries.

## Honesty rule

Only claim what you did. If an interviewer asks about a detail you did not do, say so and explain how you would approach it. That reads as integrity and is far better than being caught out.

## CV bullets (your version, ready to paste)

- Built a cloud-based SOC in Azure to monitor live cyber threats, using a honeypot VM to capture real RDP/SSH brute-force attacks.
- Investigated attack patterns in Microsoft Sentinel using KQL, visualised activity on geographic heat maps, and automated alerting with Logic Apps.
- Ran Nessus vulnerability scans and remediated misconfigurations to align infrastructure with IT security best practice.

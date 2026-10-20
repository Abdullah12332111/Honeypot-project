# 03 - KQL Analysis and Heat Map

All queries are also saved as `.kql` files in [`/queries`](../queries). Run them in Sentinel under **Logs**.

## Useful event IDs (Windows)

| Event ID | Meaning |
|---|---|
| 4625 | Failed logon |
| 4624 | Successful logon |
| 4672 | Special privileges assigned (admin logon) |
| 4688 | New process created |

Logon type **10** = RemoteInteractive (RDP). Logon type **3** = network (often NLA).

## 1. Volume of failed RDP logons

```kql
SecurityEvent
| where EventID == 4625
| summarize FailedAttempts = count() by bin(TimeGenerated, 1h)
| render timechart
```

## 2. Top attacking IPs

```kql
SecurityEvent
| where EventID == 4625 and isnotempty(IpAddress) and IpAddress != "-"
| summarize Attempts = count(), Accounts = dcount(TargetUserName) by IpAddress
| top 20 by Attempts desc
```

## 3. Most targeted usernames

```kql
SecurityEvent
| where EventID == 4625
| summarize Attempts = count() by TargetUserName
| top 20 by Attempts desc
```

Typical results are names like `administrator`, `admin`, `user`, `test`. Record yours in the report.

## 4. Linux SSH brute force

```kql
Syslog
| where Facility in ("auth", "authpriv")
| where SyslogMessage has "Failed password" or SyslogMessage has "Invalid user"
| extend SourceIP = extract(@"from (\d{1,3}(?:\.\d{1,3}){3})", 1, SyslogMessage)
| extend User = extract(@"(?:invalid user|for) (\S+)", 1, SyslogMessage)
| summarize Attempts = count() by SourceIP, User
| top 20 by Attempts desc
```

## 5. Geolocation enrichment

Using the built-in function:

```kql
SecurityEvent
| where EventID == 4625 and isnotempty(IpAddress) and IpAddress != "-"
| extend Geo = geo_info_from_ip_address(IpAddress)
| extend Country = tostring(Geo.country),
         City = tostring(Geo.city),
         Latitude = toreal(Geo.latitude),
         Longitude = toreal(Geo.longitude)
| summarize Attempts = count() by IpAddress, Country, City, Latitude, Longitude
| order by Attempts desc
```

Alternative with a GeoIP watchlist (if you import a CSV into a Sentinel Watchlist called `geoip`):

```kql
let GeoIP = _GetWatchlist("geoip");
SecurityEvent
| where EventID == 4625
| evaluate ipv4_lookup(GeoIP, IpAddress, network)
| summarize Attempts = count() by IpAddress, countryname, cityname, latitude, longitude
```

## 6. Attack heat map (Workbook)

1. In Sentinel open **Workbooks > Add workbook > Edit**.
2. **Add query**, paste the geolocation query (query 5).
3. Set **Visualization** to **Map**.
4. Under map settings: Location info using **Latitude/Longitude**, Size by **Attempts**, Metric label `Country`.
5. Save as `Honeypot Attack Map` and screenshot it.

## 7. Compromise check (important)

A brute force that succeeds is the real incident. Always check for a success after many failures from the same IP:

```kql
let Failed = SecurityEvent
    | where EventID == 4625
    | summarize Fails = count() by IpAddress, TargetUserName;
SecurityEvent
| where EventID == 4624 and LogonType in (3, 10)
| join kind=inner Failed on IpAddress, TargetUserName
| where Fails > 10
| project TimeGenerated, IpAddress, TargetUserName, LogonType, Fails
```

## 8. Spray vs brute force

- **Brute force**: many passwords against one account.
- **Password spray**: few passwords against many accounts.

```kql
SecurityEvent
| where EventID == 4625
| summarize Attempts = count(), DistinctUsers = dcount(TargetUserName) by IpAddress
| extend Pattern = case(DistinctUsers > 10 and Attempts / DistinctUsers < 5, "Likely spray",
                        DistinctUsers <= 3 and Attempts > 50, "Likely brute force",
                        "Mixed / other")
| order by Attempts desc
```

## What to write down

For the report, capture: total attempts, unique IPs, top 5 countries, top 10 usernames, time of first attack after exposure, and any successful logon. Use [06-findings-report-template.md](06-findings-report-template.md).

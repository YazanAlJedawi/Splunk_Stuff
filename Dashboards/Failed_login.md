# Security Login Activity Dashboard Overview

![splunk_screen](sceenshots\failed_login.png)

## Executive Summary: Importance and Purpose
This dashboard serves as a critical operational security and IT monitoring tool, providing a holistic, real-time view of authentication success and failure across the monitored environment. Its primary importance lies in its ability to `facilitate immediate threat detection and resource management`.

By consolidating key performance indicators (KPIs) and granular behavioral data into a single pane of glass, it allows security analysts to move from high-level volume alerts (e.g., total failures) to deep-dive investigations (e.g., who is failing, and from where). Rapid identification of unusual login patterns—whether due to brute-force attacks, credential stuffing, or simply persistent user error—is essential for maintaining network integrity and ensuring compliance. This dashboard transforms raw log data into actionable intelligence, reducing investigation time and potential exposure risk.

## Dashboard Sections and Search Processing Language (SPL)
The dashboard is logically divided into three primary rows, moving from high-level summaries (KPIs) to specific user and department activity (Charts), and concluding with granular, searchable event data (Table).

### 1. Key Performance Indicators (KPIs)
This top row provides immediate, high-level context on overall login activity volume and health. These metrics are typically displayed as Single Value visualizations.

[*] Total Logins

Purpose: The sum of all authentication attempts (both success and failure) within the time frame.

SPL: 
```bash
index="botsv3" 
| stats count as "Total Logins"

```

[*] Successful Logins

Purpose: The total volume of successful authentication events.

SPL: 
```bash
index="botsv3" status="success" 
| stats count as "Success Logins"

```
[*] Failed Logins

Purpose: The total volume of unsuccessful authentication attempts. This is a primary indicator of potential issues.

SPL: 
```bash
index="botsv3" status="Failed" 
| stats count as "Failed Logins"

```
[*] Failure Rate

Purpose: The percentage of failed logins relative to total attempts, providing a normalized measure of system security health.

SPL: 
```bash
index=""botsv3" action=login | stats count(eval(login_status="failure"))  
| eval "Failure Rate" = round((Failed / Total) * 100, 2) | fields "Failure Rate"
```

### 2. Activity Breakdown by Group and User (Bar Charts)
This row segments the failure data to identify organizational areas and individual users experiencing the most issues, facilitating targeted outreach or investigation. These metrics are typically displayed as Bar Charts.

[*] Failed Logins by Department

Purpose: Shows which organizational units generate the highest volume of failed logins, helping to identify potential training needs or targeted attacks against specific teams.

SPL: 
```bash
index="botsv3" action=login login_status=failure | stats count by department | sort -count
```
[*] Failed Logins by Top 10 Users

Purpose: Pinpoints the individual users associated with the most failed attempts, crucial for investigating account lockouts, compromised accounts, or brute-force testing.

SPL: 
```bash
index=logins action=login login_status=failure | stats count by user | sort -count | head 10
```

### 3. Detailed Event Activity (Table)
This section provides the underlying source data necessary for drilling down into specific events, serving as the launching pad for detailed security investigations. This is typically displayed as a Statistics Table.

[*] Detailed Login Events Table

Purpose: Presents granular, aggregated data per user, showing not just login counts but associated metadata like source IP counts (num_src_ip), destination systems (dest_name), and geographic location (src_country), which helps determine the scope and nature of the login activity.

SPL: 
```bash
index=botsv3 sourcetype="auth*" status="failed" 
| lookup geoip src_ip AS src_ip OUTPUT country AS src_country 
| lookup device_mapping.csv src_computer AS src_computer OUTPUT dest_name 
| table user, activity_count, src_ip, dest_name, src_computer, src_country 
| sort -activity_count
```

Y.
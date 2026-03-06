# HEARTBEAT.md — [PERSONA NAME] Scheduled Tasks

*Proactive monitoring and maintenance tasks for the [PERSONA NAME] persona.*

## Overview

This heartbeat defines recurring tasks that the persona runs automatically on a schedule. On Zo Computer, these are implemented as **Zo Agents** (Settings > Agents). On other platforms, implement via cron, scheduled workflows, or your platform's task scheduler.

## Schedule

| Task | Frequency | Priority | Delivery |
|------|-----------|----------|----------|
| [Health Check] | Every hour | High | [email/sms/none] |
| [Data Refresh] | Every 6 hours | Medium | none |
| [Daily Summary] | Daily 8:00 AM | Medium | email |
| [Weekly Review] | Weekly Monday 9:00 AM | Low | email |

## Tasks

### 1. Health Check
**Frequency:** Hourly
**Purpose:** Verify core systems and data sources are operational

**Actions:**
- [ ] Check API connectivity for [domain data sources]
- [ ] Verify data freshness (last update within expected window)
- [ ] Test critical skill scripts return valid output
- [ ] Log results to [log location]

**Alert Conditions:**
- API returns errors for >2 consecutive checks
- Data is stale by >2x the expected refresh interval
- Any skill script exits with non-zero code

**Zo Agent rrule:** `FREQ=HOURLY;INTERVAL=1`

---

### 2. Data Refresh
**Frequency:** Every 6 hours
**Purpose:** Update cached/local data from external sources

**Actions:**
- [ ] Fetch latest data from [source 1]
- [ ] Update local cache/database
- [ ] Validate data integrity (row counts, schema, checksums)
- [ ] Prune expired cache entries

**Zo Agent rrule:** `FREQ=HOURLY;INTERVAL=6`

---

### 3. Daily Summary
**Frequency:** Daily at 8:00 AM user timezone
**Purpose:** Proactive briefing on key metrics and status

**Actions:**
- [ ] Compile key metrics from the last 24 hours
- [ ] Flag any anomalies or threshold breaches
- [ ] List pending items requiring user attention
- [ ] Summarize actions taken by automated systems

**Delivery:** Email to user with formatted report

**Zo Agent rrule:** `FREQ=DAILY;BYHOUR=8;BYMINUTE=0`

---

### 4. Weekly Review
**Frequency:** Weekly on Monday at 9:00 AM
**Purpose:** Trend analysis and strategic recommendations

**Actions:**
- [ ] Aggregate weekly metrics and compare to prior week
- [ ] Identify trends (improving, declining, stable)
- [ ] Generate recommendations for the coming week
- [ ] Archive completed tasks and clean up workspace

**Delivery:** Email with PDF attachment

**Zo Agent rrule:** `FREQ=WEEKLY;BYDAY=MO;BYHOUR=9;BYMINUTE=0`

---

## Zo Agent Implementation

Create agents via Zo chat or the API:

```
Create a Zo Agent that runs every hour to check [data source] health.
If any check fails, send me an SMS. Otherwise, log results silently.
```

Or via CLI:
```bash
# Hourly health check (no notification on success)
zo agent create --rrule "FREQ=HOURLY;INTERVAL=1" \
  --instruction "Run health check for [persona]: verify [data source] API responds, check data freshness, test skill scripts. Only notify on failure." \
  --delivery none

# Daily summary via email
zo agent create --rrule "FREQ=DAILY;BYHOUR=8;BYMINUTE=0" \
  --instruction "Generate daily summary for [persona]: compile 24h metrics, flag anomalies, list pending items. Send formatted report." \
  --delivery email
```

## Stall Detection

For long-running tasks within heartbeat actions, use the heartbeat-wrapper skill:

```bash
bun /home/workspace/Skills/heartbeat-wrapper/scripts/wrapper.ts \
  "Data refresh" --timeout=120 --checkpoint \
  -- bun /home/workspace/Skills/[persona]-skill/scripts/refresh.ts
```

## Escalation

| Severity | Condition | Action |
|----------|-----------|--------|
| **Low** | Data slightly stale, non-critical metric drift | Log only |
| **Medium** | API errors, data gaps, threshold breach | Email notification |
| **High** | System down, critical data missing, security event | SMS + email |

---

*Heartbeat tasks complement the persona's interactive capabilities with proactive monitoring.*

<!-- CUSTOMIZATION NOTES:
     - Schedule: Adjust frequencies to match your domain's data refresh patterns
     - Tasks: Replace placeholders with your actual health checks, data sources, and metrics
     - Zo Agent rrules: Use RFC 5545 RRULE syntax — test with "list agents" after creation
     - Delivery: Choose email, sms, telegram, or none per task severity
     - Stall Detection: The heartbeat-wrapper skill is optional but recommended for reliability
     - Escalation: Define clear severity levels so automated alerts don't cause alert fatigue
     - On non-Zo platforms: Map rrules to cron syntax or your scheduler's format
-->

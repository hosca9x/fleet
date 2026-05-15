# opsx status

Show the current operational status of Fleet infrastructure components, services, and recent deployments.

## Usage

```
/opsx status [component] [--env <environment>] [--verbose]
```

## Arguments

- `component` (optional): Specific component to check. One of: `fleet`, `mysql`, `redis`, `s3`, `smtp`, `all` (default: `all`)
- `--env`: Target environment. One of: `production`, `staging`, `dev` (default: inferred from context)
- `--verbose`: Include detailed diagnostics and recent logs

## What This Command Does

1. **Collects service health** from each infrastructure component
2. **Checks recent deployments** and their rollout status
3. **Summarizes active alerts** from monitoring systems
4. **Reports database connectivity** and replica lag
5. **Validates cache availability** and hit rates
6. **Checks certificate expiry** for TLS endpoints

## Output Format

The agent will produce a structured status report:

```
## Fleet Operational Status — <environment> (<timestamp>)

### Core Services
| Service     | Status  | Uptime  | Version  | Notes                     |
|-------------|---------|---------|----------|---------------------------|
| fleet-web   | ✅ OK   | 14d 3h  | v4.52.1  | 3 replicas healthy        |
| fleet-worker| ✅ OK   | 14d 3h  | v4.52.1  | processing queue normally |
| mysql-primary| ✅ OK  | 30d 1h  | 8.0.35   | replica lag: 0.2s         |
| redis        | ✅ OK  | 30d 1h  | 7.2.4    | memory: 42% / 8GB         |

### Recent Deployments (last 7 days)
- v4.52.1 → production  (2 days ago) — ✅ successful, no rollback
- v4.52.0 → staging     (5 days ago) — ✅ successful

### Active Alerts
- ⚠️  MySQL replica lag exceeded 1s threshold (resolved 3h ago)
- No current active alerts

### Certificate Expiry
- fleet.example.com: expires in 87 days
```

## Agent Instructions

When this command is invoked, you are acting as an SRE performing a health check. Follow these steps:

### Step 1: Determine Environment

If `--env` is not specified:
- Check for environment indicators in the conversation (e.g., recent deploy context, mentioned hostnames)
- Ask the user to confirm if ambiguous: "Which environment should I check — production, staging, or dev?"
- Default to `staging` if truly unknown to avoid accidentally surfacing sensitive production details

### Step 2: Gather Status Information

For each requested component, collect:

**Fleet Application**
- Number of healthy/unhealthy replicas
- Current deployed version
- Uptime of oldest replica
- Recent error rate from logs (5xx responses in last 15 min)
- Active websocket/live query connections

**MySQL**
- Primary connectivity
- Replica count and individual lag values
- Slow query count in last hour
- Disk usage percentage
- Active connection count vs max_connections

**Redis**
- Connectivity and role (primary/replica)
- Memory usage vs maxmemory
- Keyspace hit rate
- Connected clients

**S3 / Object Storage**
- Bucket accessibility (software installers, carve results)
- Recent upload/download error rates

**SMTP / Email**
- Last successful email delivery timestamp
- Bounce rate if available

### Step 3: Check Deployment History

Look up the last 5 deployments:
- Version deployed
- Target environment
- Timestamp and duration
- Success/failure status
- Any associated rollbacks

### Step 4: Summarize Alerts

- List any currently firing alerts with severity
- Note recently resolved alerts (last 24h) that may still be relevant
- Highlight if any alert has been firing for >1 hour without acknowledgment

### Step 5: Format and Present

- Use the table format shown above for quick scanning
- Use ✅ for healthy, ⚠️ for degraded/warning, ❌ for critical/down
- Keep the summary concise; use `--verbose` output for deep-dive details
- If `--verbose` is set, append recent relevant log lines (last 20 lines of errors) per component

## Examples

**Quick overall check:**
```
/opsx status
```

**Check only database in production:**
```
/opsx status mysql --env production
```

**Full verbose status for staging:**
```
/opsx status all --env staging --verbose
```

## Related Commands

- `/opsx deploy` — Deploy a new version
- `/opsx diff` — Show what changed between versions
- `/opsx apply` — Apply infrastructure changes
- `/opsx archive` — Archive old data or resources

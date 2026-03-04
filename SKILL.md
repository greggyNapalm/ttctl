# ttctl — tt HTTP API CLI

CLI for managing the tt monitoring platform. Covers teams, monitors, probes, alert channels, checkers, users, and more.

## Setup

The CLI must be authenticated before use. Two methods:

**Device flow (interactive):**
```bash
ttctl login --server-url https://tt.example.com
```

**Static token (CI/scripting):**
```bash
export TT_SERVER_URL=https://tt.example.com
export TT_TOKEN=<jwt>
```

Verify auth: `ttctl whoami`

## Global Flags

| Flag | Env var | Description |
|------|---------|-------------|
| `--server-url` | `TT_SERVER_URL` | tt HTTP API base URL |
| `--token` | `TT_TOKEN` | Static Bearer token |
| `--profile` | `TT_PROFILE` | Config profile name |
| `--json` | — | JSON output (default: table) |
| `--no-color` | `NO_COLOR` | Disable colors |
| `--verbose` | — | Print HTTP details |

## Command Reference

### Auth

| Command | Description |
|---------|-------------|
| `ttctl login --server-url <url>` | Authenticate via device flow |
| `ttctl logout` | Remove cached token |
| `ttctl whoami` | Show current user (id, email, name, role) |
| `ttctl version` | Print version and build info |

### Teams

| Command | Description |
|---------|-------------|
| `ttctl teams list` | List teams |
| `ttctl teams get <id>` | Get team details |
| `ttctl teams create --name <name>` | Create team |
| `ttctl teams update <id> --name <name>` | Rename team |
| `ttctl teams delete <id>` | Delete team |

### Members

All require `--team <id>`.

| Command | Description |
|---------|-------------|
| `ttctl members list --team <id>` | List members |
| `ttctl members add --team <id> --user <uid> [--role admin\|member]` | Add member |
| `ttctl members remove <userID> --team <id>` | Remove member |
| `ttctl members set-role <userID> --team <id> --role <role>` | Change role (owner/admin/member) |

### Invites

All require `--team <id>` except accept.

| Command | Description |
|---------|-------------|
| `ttctl invites list --team <id>` | List pending invites |
| `ttctl invites create --team <id>` | Create invite link |
| `ttctl invites revoke <inviteID> --team <id>` | Revoke invite |
| `ttctl invites accept <token>` | Accept invite (no --team needed) |

### Monitors

All require `--team <id>`.

| Command | Description |
|---------|-------------|
| `ttctl monitors list --team <id>` | List monitors |
| `ttctl monitors get <monitorID> --team <id>` | Get monitor details |
| `ttctl monitors create --team <id> --name <n> --url <u> [flags]` | Create monitor |
| `ttctl monitors update <monitorID> --team <id> [flags]` | Update monitor |
| `ttctl monitors delete <monitorID> --team <id>` | Delete monitor |
| `ttctl monitors pause <monitorID> --team <id>` | Pause monitoring |
| `ttctl monitors unpause <monitorID> --team <id>` | Resume monitoring |
| `ttctl monitors duplicate <monitorID> --team <id>` | Clone monitor |
| `ttctl monitors metrics <monitorID> --team <id>` | Uptime%, MTBF, MTTR |
| `ttctl monitors expiration <monitorID> --team <id>` | SSL/domain expiry |

**Monitor create/update flags:**

| Flag | Type | Description |
|------|------|-------------|
| `--name` | string | Display name |
| `--url` | string | Target URL |
| `--timeout` | int | Timeout in ms (default 10000) |
| `--interval` | int | Check interval in ms (default 60000, min 30000) |
| `--method` | string | HTTP method (default GET) |
| `--http-version` | string | Force HTTP version |
| `--header` | key=value | HTTP header (repeatable) |
| `--debug` | bool | Enable debug probes |
| `--checker-region` | string | Preferred region |
| `--checker-country` | string | Preferred country |
| `--checker-id` | string | Pin to checker |
| `--allowed-status` | []int | Accepted status codes (repeatable) |
| `--latency-threshold` | int | Max latency in ms |
| `--failure-threshold` | int | Failures before down |
| `--ssl-expiry` | bool | SSL expiry alerts (default true) |
| `--ssl-expiry-thresholds` | []int | SSL alert thresholds in days (comma-separated) |
| `--domain-expiry` | bool | Domain expiry alerts (default true) |
| `--domain-expiry-thresholds` | []int | Domain alert thresholds in days (comma-separated) |

### Probes

All require `--team <id> --monitor <id>`.

| Command | Description |
|---------|-------------|
| `ttctl probes list --team <id> --monitor <id> [--from] [--to] [--status] [--limit] [--before\|--after]` | List probes (with optional time/status/pagination filters) |
| `ttctl probes get <probeID> --team <id> --monitor <id>` | Get probe |
| `ttctl probes create --team <id> --monitor <id> [flags]` | Create probe |
| `ttctl probes update <probeID> --team <id> --monitor <id> [flags]` | Update probe |
| `ttctl probes delete <probeID> --team <id> --monitor <id>` | Delete probe |
| `ttctl probes aggregate --team <id> --monitor <id> --from <epoch> --to <epoch> --step <dur> --agg <fn>` | Time-series aggregation |

**Probes list filter flags:**

| Flag | Type | Description |
|------|------|-------------|
| `--from` | int (Unix epoch) | Start of time range |
| `--to` | int (Unix epoch) | End of time range |
| `--status` | int | Filter by HTTP status code |
| `--limit` | int | Page size (max 100) |
| `--before` | RFC 3339 | Cursor for newest-first pagination |
| `--after` | RFC 3339 | Cursor for oldest-first pagination |

**Probes aggregate flags (all required):**

| Flag | Type | Values |
|------|------|--------|
| `--from` | int (Unix epoch) | Start of range |
| `--to` | int (Unix epoch) | End of range |
| `--step` | string | `1m`, `5m`, `15m`, `30m`, `1h`, `3h`, `6h`, `12h`, `1d` |
| `--agg` | string | `avg`, `median`, `min`, `max`, `p50`, `p75`, `p90`, `p95`, `p99` |

### Alert Channels

All require `--team <id>`.

| Command | Description |
|---------|-------------|
| `ttctl alert-channels list --team <id>` | List team channels |
| `ttctl alert-channels create --team <id> --type <t> --name <n> --config '<json>'` | Create channel |
| `ttctl alert-channels update <channelID> --team <id> [--name] [--type] [--config] [--enabled]` | Update channel |
| `ttctl alert-channels delete <channelID> --team <id>` | Delete channel |

Channel types: `slack`, `webhook`, `email`, `telegram`, `discord`.

### Personal Alert Channels (my-channels)

All require `--team <id>`.

| Command | Description |
|---------|-------------|
| `ttctl my-channels list --team <id>` | List personal channels |
| `ttctl my-channels create --team <id> --type <t> --name <n> --config '<json>'` | Create |
| `ttctl my-channels update <id> --team <id> [flags]` | Update |
| `ttctl my-channels delete <id> --team <id>` | Delete |

### Monitor Alert Channel Links (monitor-channels)

All require `--team <id> --monitor <id>`.

| Command | Description |
|---------|-------------|
| `ttctl monitor-channels list --team <id> --monitor <id>` | List linked channels |
| `ttctl monitor-channels set --team <id> --monitor <id> --channel <c1> [--channel <c2>]` | Replace all links |
| `ttctl monitor-channels add <channelID> --team <id> --monitor <id>` | Link channel |
| `ttctl monitor-channels remove <channelID> --team <id> --monitor <id>` | Unlink channel |

### Alerts

| Command | Description |
|---------|-------------|
| `ttctl alerts list --team <id> --monitor <id>` | List alert events |

### Checkers

| Command | Description |
|---------|-------------|
| `ttctl checkers list` | List all checkers |
| `ttctl checkers get <id>` | Get checker details |
| `ttctl checkers rename <id> --name <name>` | Rename checker |
| `ttctl checkers delete <id>` | Delete checker |
| `ttctl checkers assignments <id>` | Show monitor assignments |
| `ttctl checkers set-teams <id> --team-ids <id1,id2,...>` | Set allowed teams |
| `ttctl checkers revoke <id>` | Revoke token |
| `ttctl checkers disconnect <id>` | Force disconnect |
| `ttctl checkers redirect <id> --url <url>` | Set redirect URL |
| `ttctl checkers clear-redirect <id>` | Clear redirect |

### Users (admin only)

| Command | Description |
|---------|-------------|
| `ttctl users list` | List users |
| `ttctl users get <id>` | Get user |
| `ttctl users create --email <e> --name <n>` | Create user |
| `ttctl users update <id> [--email] [--name]` | Update user |
| `ttctl users delete <id>` | Delete user |

### Preferences

| Command | Description |
|---------|-------------|
| `ttctl preferences get` | Get preferences |
| `ttctl preferences set [--time-format 12h\|24h] [--theme light\|dark]` | Set preferences |

### Geo

| Command | Description |
|---------|-------------|
| `ttctl geo regions` | List checker regions |

### Telegram

Requires `--team <id>`.

| Command | Description |
|---------|-------------|
| `ttctl telegram create-link --team <id>` | Create Telegram linking code |
| `ttctl telegram link-status <code> --team <id>` | Check link status |

## Output Modes

**Table (default):** Human-readable columns. Example:

```
ID                                    NAME            STATUS   HEALTH
550e8400-e29b-41d4-a716-446655440000  Production API  active   up
660e8400-e29b-41d4-a716-446655440001  Staging API     paused   unknown
```

**JSON (`--json`):** Raw API response. Pipe to `jq` for filtering:

```bash
ttctl monitors list --team $T --json | jq '.[].name'
```

## Common Workflows

### Create a monitor with alert channels

```bash
TEAM=$(ttctl teams list --json | jq -r '.[0].id')

# Create a Slack alert channel
CHAN=$(ttctl alert-channels create --team $TEAM \
  --type slack \
  --name "Ops Alerts" \
  --config '{"webhook_url":"https://hooks.slack.com/services/..."}' \
  --json | jq -r '.id')

# Create a monitor
MON=$(ttctl monitors create --team $TEAM \
  --name "Production API" \
  --url https://api.example.com/health \
  --interval 60000 \
  --failure-threshold 3 \
  --json | jq -r '.id')

# Link channel to monitor
ttctl monitor-channels add $CHAN --team $TEAM --monitor $MON
```

### Investigate downtime

```bash
TEAM=<team-id>
MON=<monitor-id>

# Check current health and status
ttctl monitors get $MON --team $TEAM

# View uptime metrics
ttctl monitors metrics $MON --team $TEAM

# List recent probes
ttctl probes list --team $TEAM --monitor $MON

# Check alert history
ttctl alerts list --team $TEAM --monitor $MON

# Check SSL/domain expiration
ttctl monitors expiration $MON --team $TEAM
```

### Onboard a new team member

```bash
TEAM=<team-id>

# Create an invite
ttctl invites create --team $TEAM

# (Share the invite link with the new member)
# The new member runs:
ttctl invites accept <token>

# Set their role to admin
ttctl members set-role <user-id> --team $TEAM --role admin
```

### Bulk-pause all monitors

```bash
TEAM=<team-id>
for id in $(ttctl monitors list --team $TEAM --json | jq -r '.[].id'); do
  ttctl monitors pause "$id" --team $TEAM
done
```

### Checker fleet overview

```bash
# List all checkers with online status
ttctl checkers list

# View assignments for a specific checker
ttctl checkers assignments <checker-id>

# Check available regions
ttctl geo regions
```

## Error Handling

| Exit code | Meaning |
|-----------|---------|
| 0 | Success |
| 1 | General error (invalid arguments, unexpected failure) |
| 2 | Authentication error (not logged in, token expired, run `ttctl login`) |
| 3 | Authorization error (forbidden, insufficient role) |
| 4 | Resource not found |
| 5 | Validation error (invalid input, 422 response) |

Errors print to stderr. JSON mode also outputs errors as JSON:

```json
{"error": "monitor not found", "code": 4}
```

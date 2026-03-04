# ttctl — tt HTTP API CLI

CLI for managing the tt monitoring platform.

## Setup

The CLI must be authenticated before use. Two methods:

**Device flow (interactive):**
```bash
ttctl login
```

**Static token (CI/scripting):**
```bash
export TT_TOKEN=<jwt>
```

The default server URL is `https://tt.myddns.me`. Override with `--server-url` or `TT_SERVER_URL`.

Verify auth: `ttctl whoami`

## Global Flags

| Flag | Env var | Description |
|------|---------|-------------|
| `--server-url` | `TT_SERVER_URL` | tt HTTP API base URL (default: `https://tt.myddns.me`) |
| `--token` | `TT_TOKEN` | Static Bearer token |
| `--profile` | `TT_PROFILE` | Config profile name |
| `--json` | — | JSON output (default: table) |
| `--no-color` | `NO_COLOR` | Disable colors |
| `--verbose` | — | Print HTTP request/response details |

## Command Reference

### Auth

| Command | Description |
|---------|-------------|
| `ttctl login` | Authenticate via device flow |
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
| `ttctl members list --team <id>` | List team members |
| `ttctl members add --team <id> --user <uid> [--role member]` | Add member |
| `ttctl members remove <userID> --team <id>` | Remove member |
| `ttctl members set-role <userID> --team <id> --role <role>` | Change role |

### Invites

`--team` required for list/create/revoke; not needed for accept.

| Command | Description |
|---------|-------------|
| `ttctl invites list --team <id>` | List invites |
| `ttctl invites create --team <id>` | Create invite |
| `ttctl invites revoke <inviteID> --team <id>` | Revoke invite |
| `ttctl invites accept <token>` | Accept invite (no --team) |

### Users (admin)

| Command | Description |
|---------|-------------|
| `ttctl users list` | List all users |
| `ttctl users get <id>` | Get user details |
| `ttctl users create --email <e> --name <n>` | Create user |
| `ttctl users update <id> [--email <e>] [--name <n>]` | Update user |
| `ttctl users delete <id>` | Delete user |

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
| `ttctl probes list --team <id> --monitor <id> [flags]` | List probes |
| `ttctl probes get <probeID> --team <id> --monitor <id>` | Get probe details |
| `ttctl probes create --team <id> --monitor <id> --status <n> [flags]` | Create probe |
| `ttctl probes update <probeID> --team <id> --monitor <id> [flags]` | Update probe |
| `ttctl probes delete <probeID> --team <id> --monitor <id>` | Delete probe |
| `ttctl probes aggregate --team <id> --monitor <id> --from <ts> --to <ts> --step <s> --agg <a>` | Aggregate |

**Probe list flags:** `--from`, `--to` (unix timestamps), `--status`, `--limit`, `--before`, `--after` (cursors).

**Probe create flags:** `--status` (required), `--total-ms`, `--dns-ms`, `--connect-ms`, `--tls-ms`, `--first-byte-ms`, `--transfer-ms`, `--error`, `--checked-at`.

### Alerts

Require `--team <id> --monitor <id>`.

| Command | Description |
|---------|-------------|
| `ttctl alerts list --team <id> --monitor <id>` | List alert events |

### Alert Channels

Require `--team <id>`.

| Command | Description |
|---------|-------------|
| `ttctl alert-channels list --team <id>` | List channels |
| `ttctl alert-channels create --team <id> --type <t> --name <n> --config '<json>'` | Create |
| `ttctl alert-channels update <channelID> --team <id> [flags]` | Update |
| `ttctl alert-channels delete <channelID> --team <id>` | Delete |

**Create/update flags:** `--type`, `--name`, `--config` (JSON string), `--enabled`.

### My Channels (personal)

| Command | Description |
|---------|-------------|
| `ttctl my-channels list` | List personal channels |
| `ttctl my-channels create --type <t> --name <n> --config '<json>'` | Create |
| `ttctl my-channels update <channelID> [flags]` | Update |
| `ttctl my-channels delete <channelID>` | Delete |

### Monitor Channels

Require `--team <id> --monitor <id>`.

| Command | Description |
|---------|-------------|
| `ttctl monitor-channels list --team <id> --monitor <id>` | List linked channels |
| `ttctl monitor-channels set --team <id> --monitor <id> --channel <id>[,...]` | Replace all links |
| `ttctl monitor-channels add <channelID> --team <id> --monitor <id>` | Link channel |
| `ttctl monitor-channels remove <channelID> --team <id> --monitor <id>` | Unlink channel |

### Checkers

No `--team` flag needed.

| Command | Description |
|---------|-------------|
| `ttctl checkers list` | List checkers |
| `ttctl checkers get <id>` | Get checker details |
| `ttctl checkers rename <id> --name <n>` | Rename checker |
| `ttctl checkers delete <id>` | Delete checker |
| `ttctl checkers assignments <id>` | List monitor assignments |
| `ttctl checkers set-teams <id> --team-ids <csv>` | Set team associations |
| `ttctl checkers revoke <id>` | Revoke registration |
| `ttctl checkers disconnect <id>` | Force disconnect |
| `ttctl checkers redirect <id> --url <u>` | Set redirect URL |
| `ttctl checkers clear-redirect <id>` | Clear redirect |

### Preferences

| Command | Description |
|---------|-------------|
| `ttctl preferences get` | Show preferences |
| `ttctl preferences set [--time-format 12h\|24h] [--theme light\|dark]` | Update preferences |

### Geo

| Command | Description |
|---------|-------------|
| `ttctl geo regions` | List available regions and countries |

### Telegram

Require `--team <id>`.

| Command | Description |
|---------|-------------|
| `ttctl telegram create-link --team <id>` | Create Telegram link session |
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

### Create a monitor

```bash
TEAM=$(ttctl teams list --json | jq -r '.[0].id')

ttctl monitors create --team $TEAM \
  --name "Production API" \
  --url https://api.example.com/health \
  --interval 60000 \
  --failure-threshold 3
```

### Set up alert channels

```bash
TEAM=<team-id>
MON=<monitor-id>

# Create a Slack channel
ttctl alert-channels create --team $TEAM \
  --type slack --name "Ops Slack" \
  --config '{"webhook_url":"https://hooks.slack.com/..."}'

# Link it to a monitor
CH=$(ttctl alert-channels list --team $TEAM --json | jq -r '.[0].id')
ttctl monitor-channels add $CH --team $TEAM --monitor $MON
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
ttctl probes list --team $TEAM --monitor $MON --limit 10

# Check alert history
ttctl alerts list --team $TEAM --monitor $MON
```

### Manage checkers

```bash
# List all checkers
ttctl checkers list

# View assignments for a checker
ttctl checkers assignments <checker-id>

# Rename a checker
ttctl checkers rename <checker-id> --name "US-East Checker"
```

### Bulk-pause all monitors

```bash
TEAM=<team-id>
for id in $(ttctl monitors list --team $TEAM --json | jq -r '.[].id'); do
  ttctl monitors pause "$id" --team $TEAM
done
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

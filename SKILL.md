# ttctl — tt HTTP API CLI

CLI for managing the tt monitoring platform. Currently covers auth, teams, and monitors (Phase 1).

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

### Investigate downtime

```bash
TEAM=<team-id>
MON=<monitor-id>

# Check current health and status
ttctl monitors get $MON --team $TEAM

# View uptime metrics
ttctl monitors metrics $MON --team $TEAM

# Check SSL/domain expiration
ttctl monitors expiration $MON --team $TEAM
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

## Not Yet Implemented

The following commands are planned for Phase 2+ and are **not available** in the current build:

`probes`, `alerts`, `alert-channels`, `my-channels`, `monitor-channels`, `members`, `invites`, `checkers`, `users`, `preferences`, `geo`, `telegram`

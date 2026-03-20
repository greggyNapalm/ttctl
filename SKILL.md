# oackctl — Oack CLI

CLI for managing the [Oack](https://oack.io) uptime monitoring platform.

## Setup

The CLI must be authenticated before use. Two methods:

**Device flow (interactive):**
```bash
oackctl login
```

**Static token (CI/scripting):**
```bash
export TT_TOKEN=<jwt>
```

The default server URL is `https://api.oack.io`. Override with `--server-url` or `TT_SERVER_URL`.

Verify auth: `oackctl whoami`

## Global Flags

| Flag | Env var | Description |
|------|---------|-------------|
| `--server-url` | `TT_SERVER_URL` | Oack API base URL (default: `https://api.oack.io`) |
| `--token` | `TT_TOKEN` | Static Bearer token |
| `--profile` | `TT_PROFILE` | Config profile name |
| `--json` | — | JSON output (default: table) |
| `--no-color` | `NO_COLOR` | Disable colors |
| `--verbose` | — | Print HTTP request/response details |

## Command Reference

### Auth

| Command | Description |
|---------|-------------|
| `oackctl login` | Authenticate via device flow |
| `oackctl logout` | Remove cached token |
| `oackctl whoami` | Show current user (id, email, name, role) |
| `oackctl version` | Print version and build info |

### Accounts

| Command | Description |
|---------|-------------|
| `oackctl accounts list` | List accounts |
| `oackctl accounts get <id>` | Get account details |
| `oackctl accounts create --name <name>` | Create account |
| `oackctl accounts update <id> --name <name>` | Update account |
| `oackctl accounts delete <id>` | Delete account |
| `oackctl accounts restore <id>` | Restore soft-deleted account |
| `oackctl accounts transfer <id> --user-id <uid>` | Transfer ownership |
| `oackctl accounts members <id>` | List account members |
| `oackctl accounts set-role <accountID> <userID> --role <role>` | Set member role |
| `oackctl accounts remove-member <accountID> <userID>` | Remove member |
| `oackctl accounts subscription <id>` | View subscription |
| `oackctl accounts update-subscription <id> --plan <p> --status <s>` | Update subscription |
| `oackctl accounts create-team <id> --name <name>` | Create team under account |
| `oackctl accounts list-teams <id>` | List account teams |
| `oackctl accounts create-invite <id> --email <e> --role <r>` | Invite to account |
| `oackctl accounts list-invites <id>` | List account invites |
| `oackctl accounts revoke-invite <accountID> <inviteID>` | Revoke invite |

### Teams

| Command | Description |
|---------|-------------|
| `oackctl teams list` | List teams |
| `oackctl teams get <id>` | Get team details |
| `oackctl teams create --name <name>` | Create team |
| `oackctl teams update <id> --name <name>` | Rename team |
| `oackctl teams delete <id>` | Delete team |

### Members

All require `--team <id>`.

| Command | Description |
|---------|-------------|
| `oackctl members list --team <id>` | List team members |
| `oackctl members add --team <id> --user <uid> [--role member]` | Add member |
| `oackctl members remove <userID> --team <id>` | Remove member |
| `oackctl members set-role <userID> --team <id> --role <role>` | Change role |

### Invites

`--team` required for list/create/revoke; not needed for accept.

| Command | Description |
|---------|-------------|
| `oackctl invites list --team <id>` | List invites |
| `oackctl invites create --team <id>` | Create invite |
| `oackctl invites revoke <inviteID> --team <id>` | Revoke invite |
| `oackctl invites accept <token>` | Accept invite (no --team) |

### Monitors

All require `--team <id>`.

| Command | Description |
|---------|-------------|
| `oackctl monitors list --team <id>` | List monitors |
| `oackctl monitors get <monitorID> --team <id>` | Get monitor details |
| `oackctl monitors create --team <id> --name <n> --url <u> [flags]` | Create monitor |
| `oackctl monitors update <monitorID> --team <id> [flags]` | Update monitor |
| `oackctl monitors delete <monitorID> --team <id>` | Delete monitor |
| `oackctl monitors pause <monitorID> --team <id>` | Pause monitoring |
| `oackctl monitors unpause <monitorID> --team <id>` | Resume monitoring |
| `oackctl monitors duplicate <monitorID> --team <id>` | Clone monitor |
| `oackctl monitors move <monitorID> --team <id> --target-team <tid>` | Move to another team |
| `oackctl monitors metrics <monitorID> --team <id>` | Uptime%, MTBF, MTTR |
| `oackctl monitors expiration <monitorID> --team <id>` | SSL/domain expiry |
| `oackctl monitors test-alert <monitorID> --team <id>` | Send test alert to all linked channels |

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
| `--follow-redirects` | bool | Follow HTTP redirects |
| `--resolve-override-ip` | string | DNS resolve override IP |
| `--checker-region` | string | Preferred region |
| `--checker-country` | string | Preferred country |
| `--checker-id` | string | Pin to checker |
| `--allowed-status` | []string | Accepted status codes/patterns (repeatable, e.g. "2xx", "301") |
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
| `oackctl probes list --team <id> --monitor <id> [flags]` | List probes |
| `oackctl probes get <probeID> --team <id> --monitor <id>` | Get probe |
| `oackctl probes details <probeID> --team <id> --monitor <id>` | Full details (geo, trace, CF log, performance) |
| `oackctl probes aggregate --team <id> --monitor <id> --from <ts> --to <ts> --step <s> --agg <a>` | Aggregate |
| `oackctl probes pcap <probeID> --team <id> --monitor <id> [-o file.pcap]` | Download pcap |

**Probe list flags:** `--from`, `--to` (unix timestamps), `--status`, `--limit`, `--before`, `--after` (cursors).

### Alerts

Require `--team <id> --monitor <id>`.

| Command | Description |
|---------|-------------|
| `oackctl alerts list --team <id> --monitor <id>` | List alert events |

### Alert Channels

Require `--team <id>`.

| Command | Description |
|---------|-------------|
| `oackctl alert-channels list --team <id>` | List channels |
| `oackctl alert-channels create --team <id> --type <t> --name <n> --config '<json>'` | Create |
| `oackctl alert-channels update <channelID> --team <id> [flags]` | Update |
| `oackctl alert-channels delete <channelID> --team <id>` | Delete |
| `oackctl alert-channels test <channelID> --team <id>` | Send test notification |

**Create/update flags:** `--type`, `--name`, `--config` (JSON string), `--enabled`.

### Monitor Channels

Require `--team <id> --monitor <id>`.

| Command | Description |
|---------|-------------|
| `oackctl monitor-channels list --team <id> --monitor <id>` | List linked channels |
| `oackctl monitor-channels set --team <id> --monitor <id> --channel <id>[,...]` | Replace all links |
| `oackctl monitor-channels add <channelID> --team <id> --monitor <id>` | Link channel |
| `oackctl monitor-channels remove <channelID> --team <id> --monitor <id>` | Unlink channel |

### Notifications

| Command | Description |
|---------|-------------|
| `oackctl notifications defaults --account <id>` | View default notification channels |
| `oackctl notifications set-defaults --account <id> --channel-ids <csv>` | Set defaults |
| `oackctl notifications copy-channels --from-account <id> --to-account <id>` | Copy channels between accounts |
| `oackctl notifications monitor-override --team <id> --monitor <id>` | View monitor override |
| `oackctl notifications set-monitor-override --team <id> --monitor <id> --channel-ids <csv>` | Set override |
| `oackctl notifications remove-monitor-override --team <id> --monitor <id>` | Remove override |

### Status Pages

All require `--account <id>`. Component/incident/maintenance subcommands also require `--page <id>`.

| Command | Description |
|---------|-------------|
| `oackctl status-pages list --account <id>` | List status pages |
| `oackctl status-pages get <pageID> --account <id>` | Get status page |
| `oackctl status-pages create --account <id> --name <n> --slug <s>` | Create |
| `oackctl status-pages update <pageID> --account <id> [flags]` | Update |
| `oackctl status-pages delete <pageID> --account <id>` | Delete |
| **Components** | |
| `oackctl status-pages create-component --account <id> --page <id> --name <n>` | Create component |
| `oackctl status-pages list-components --account <id> --page <id>` | List components |
| `oackctl status-pages update-component <compID> --account <id> --page <id> [flags]` | Update |
| `oackctl status-pages delete-component <compID> --account <id> --page <id>` | Delete |
| **Component Groups** | |
| `oackctl status-pages create-group --account <id> --page <id> --name <n>` | Create group |
| `oackctl status-pages list-groups --account <id> --page <id>` | List groups |
| `oackctl status-pages update-group <groupID> --account <id> --page <id> --name <n>` | Update |
| `oackctl status-pages delete-group <groupID> --account <id> --page <id>` | Delete |
| **Incidents** | |
| `oackctl status-pages create-incident --account <id> --page <id> --name <n> --severity <s>` | Create |
| `oackctl status-pages list-incidents --account <id> --page <id>` | List |
| `oackctl status-pages get-incident <incID> --account <id> --page <id>` | Get |
| `oackctl status-pages update-incident <incID> --account <id> --page <id> [flags]` | Update |
| `oackctl status-pages delete-incident <incID> --account <id> --page <id>` | Delete |
| `oackctl status-pages post-incident-update <incID> --account <id> --page <id> --status <s> --message <m>` | Post update |
| **Maintenances** | |
| `oackctl status-pages create-maintenance --account <id> --page <id> --name <n> --scheduled-at <t> --duration <m>` | Create |
| `oackctl status-pages list-maintenances --account <id> --page <id>` | List |
| `oackctl status-pages get-maintenance <maintID> --account <id> --page <id>` | Get |
| `oackctl status-pages update-maintenance <maintID> --account <id> --page <id> [flags]` | Update |
| `oackctl status-pages delete-maintenance <maintID> --account <id> --page <id>` | Delete |
| `oackctl status-pages post-maintenance-update <maintID> --account <id> --page <id> --status <s> --message <m>` | Post update |
| **Subscribers** | |
| `oackctl status-pages list-subscribers --account <id> --page <id>` | List |
| `oackctl status-pages remove-subscriber <subID> --account <id> --page <id>` | Remove |
| **Watchdogs** | |
| `oackctl status-pages create-watchdog --account <id> --page <id> --component <cid> --monitor-id <mid>` | Create |
| `oackctl status-pages list-watchdogs --account <id> --page <id> --component <cid>` | List |
| `oackctl status-pages update-watchdog <wdID> --account <id> --page <id> --component <cid> [flags]` | Update |
| `oackctl status-pages delete-watchdog <wdID> --account <id> --page <id> --component <cid>` | Delete |
| **Incident Templates** | |
| `oackctl status-pages create-template --account <id> --page <id> --name <n> --severity <s>` | Create |
| `oackctl status-pages list-templates --account <id> --page <id>` | List |
| `oackctl status-pages update-template <tplID> --account <id> --page <id> [flags]` | Update |
| `oackctl status-pages delete-template <tplID> --account <id> --page <id>` | Delete |

### Comments

Require `--team <id> --monitor <id>`.

| Command | Description |
|---------|-------------|
| `oackctl comments create --team <id> --monitor <id> --body <text>` | Create comment |
| `oackctl comments list --team <id> --monitor <id>` | List comments |
| `oackctl comments edit <commentID> --team <id> --monitor <id> --body <text>` | Edit |
| `oackctl comments delete <commentID> --team <id> --monitor <id>` | Delete |
| `oackctl comments reply <commentID> --team <id> --monitor <id> --body <text>` | Reply |
| `oackctl comments list-replies <commentID> --team <id> --monitor <id>` | List replies |
| `oackctl comments resolve <commentID> --team <id> --monitor <id>` | Resolve thread |
| `oackctl comments reopen <commentID> --team <id> --monitor <id>` | Reopen thread |
| `oackctl comments list-edits <commentID> --team <id> --monitor <id>` | View edit history |
| `oackctl comments list-by-team --team <id>` | All comments in team |
| `oackctl comments list-by-account --account <id>` | All comments in account |

### External Links

Require `--team <id>`.

| Command | Description |
|---------|-------------|
| `oackctl external-links create --team <id> --name <n> --url <u> [--icon-url <u>]` | Create |
| `oackctl external-links list --team <id>` | List |
| `oackctl external-links get <linkID> --team <id>` | Get |
| `oackctl external-links update <linkID> --team <id> [flags]` | Update |
| `oackctl external-links delete <linkID> --team <id>` | Delete |
| `oackctl external-links assign --team <id> --monitor <id> --link-id <id>` | Assign to monitor |
| `oackctl external-links list-by-monitor --team <id> --monitor <id>` | List by monitor |
| `oackctl external-links unassign <linkID> --team <id> --monitor <id>` | Unassign |

### Chart Events

Require `--team <id>`.

| Command | Description |
|---------|-------------|
| `oackctl chart-events create --team <id> --title <t> --kind <k> --source <s> [flags]` | Create |
| `oackctl chart-events list --team <id> [--from <ts> --to <ts> --kind <k> --source <s> --monitor <id>]` | List |
| `oackctl chart-events update <eventID> --team <id> [flags]` | Update |
| `oackctl chart-events delete <eventID> --team <id>` | Delete |

### Shares

Require `--team <id> --monitor <id>`.

| Command | Description |
|---------|-------------|
| `oackctl shares create --team <id> --monitor <id> [--mode public\|authenticated] [--expires-at <t>]` | Create |
| `oackctl shares list --team <id> --monitor <id>` | List |
| `oackctl shares revoke <shareID> --team <id> --monitor <id>` | Revoke |

### Traces

Require `--team <id> --monitor <id>`.

| Command | Description |
|---------|-------------|
| `oackctl traces list --team <id> --monitor <id>` | List traces |
| `oackctl traces request --team <id> --monitor <id>` | Request new trace |

### Timeline

Require `--team <id> --monitor <id>`.

| Command | Description |
|---------|-------------|
| `oackctl timeline list --team <id> --monitor <id> [--from <ts> --to <ts> --kinds <csv>]` | List events |

### CF Logs

Require `--team <id> --monitor <id>`.

| Command | Description |
|---------|-------------|
| `oackctl cf-logs get <probeID> --team <id> --monitor <id>` | Get CF log for probe |
| `oackctl cf-logs list --team <id> --monitor <id> [--from <ts> --to <ts> --limit <n>]` | List CF logs |

### API Keys

Require `--team <id>`.

| Command | Description |
|---------|-------------|
| `oackctl api-keys list --team <id>` | List API keys (prefix only) |
| `oackctl api-keys create --team <id> --name <n>` | Create key (full key shown once) |
| `oackctl api-keys delete <keyID> --team <id>` | Delete key |

### Integrations

#### PagerDuty

Require `--account <id>`.

| Command | Description |
|---------|-------------|
| `oackctl pagerduty create --account <id> --api-key <key>` | Create integration |
| `oackctl pagerduty get --account <id>` | View integration |
| `oackctl pagerduty update --account <id> --api-key <key>` | Update |
| `oackctl pagerduty delete --account <id>` | Delete |
| `oackctl pagerduty sync --account <id>` | Force sync |

#### Cloudflare Zone

Require `--account <id>`.

| Command | Description |
|---------|-------------|
| `oackctl cf-integrations create --account <id> --zone-id <z> --api-token <t>` | Create |
| `oackctl cf-integrations list --account <id>` | List |
| `oackctl cf-integrations get <integrationID> --account <id>` | Get |
| `oackctl cf-integrations update <integrationID> --account <id> --api-token <t>` | Update |
| `oackctl cf-integrations delete <integrationID> --account <id>` | Delete |

### Devices

| Command | Description |
|---------|-------------|
| `oackctl devices register --token <pushToken> --platform <ios\|android>` | Register push device |
| `oackctl devices list` | List registered devices |
| `oackctl devices unregister <token>` | Unregister device |

### Preferences

| Command | Description |
|---------|-------------|
| `oackctl preferences get` | Show preferences |
| `oackctl preferences set [--time-format 12h\|24h] [--theme light\|dark]` | Update |

### Geo

| Command | Description |
|---------|-------------|
| `oackctl geo regions` | List available regions and countries |

### Telegram

Require `--team <id>`.

| Command | Description |
|---------|-------------|
| `oackctl telegram create-link --team <id>` | Create Telegram link session |
| `oackctl telegram link-status <code> --team <id>` | Check link status |

## Output Modes

**Table (default):** Human-readable columns. Example:

```
ID                                    NAME            STATUS   HEALTH
550e8400-e29b-41d4-a716-446655440000  Production API  active   up
660e8400-e29b-41d4-a716-446655440001  Staging API     paused   unknown
```

**JSON (`--json`):** Raw API response. Pipe to `jq` for filtering:

```bash
oackctl monitors list --team $T --json | jq '.[].name'
```

## Common Workflows

### Create a monitor with alerts

```bash
TEAM=$(oackctl teams list --json | jq -r '.[0].id')

oackctl monitors create --team $TEAM \
  --name "Production API" \
  --url https://api.example.com/health \
  --interval 60000 \
  --failure-threshold 3

# Create a Slack channel and link it
oackctl alert-channels create --team $TEAM \
  --type slack --name "Ops Slack" \
  --config '{"webhook_url":"https://hooks.slack.com/..."}'

CH=$(oackctl alert-channels list --team $TEAM --json | jq -r '.[0].id')
MON=$(oackctl monitors list --team $TEAM --json | jq -r '.[0].id')
oackctl monitor-channels add $CH --team $TEAM --monitor $MON
```

### Manage status page incidents

```bash
ACCT=<account-id>
PAGE=<page-id>

# Create an incident
oackctl status-pages create-incident --account $ACCT --page $PAGE \
  --name "API degradation" --severity major --message "Investigating high latency"

# Post an update
INC=$(oackctl status-pages list-incidents --account $ACCT --page $PAGE --json | jq -r '.[0].id')
oackctl status-pages post-incident-update $INC --account $ACCT --page $PAGE \
  --status identified --message "Root cause identified, fix in progress"

# Resolve
oackctl status-pages update-incident $INC --account $ACCT --page $PAGE --status resolved
```

### Investigate downtime

```bash
TEAM=<team-id>
MON=<monitor-id>

# Check health and metrics
oackctl monitors get $MON --team $TEAM
oackctl monitors metrics $MON --team $TEAM

# View recent probes
oackctl probes list --team $TEAM --monitor $MON --limit 10

# Get detailed probe info (geo, CF enrichment, performance percentiles)
PROBE=$(oackctl probes list --team $TEAM --monitor $MON --limit 1 --json | jq -r '.[0].id')
oackctl probes details $PROBE --team $TEAM --monitor $MON

# Check timeline for correlated events
oackctl timeline list --team $TEAM --monitor $MON

# View alert history
oackctl alerts list --team $TEAM --monitor $MON
```

### Bulk-pause all monitors

```bash
TEAM=<team-id>
for id in $(oackctl monitors list --team $TEAM --json | jq -r '.[].id'); do
  oackctl monitors pause "$id" --team $TEAM
done
```

## Error Handling

| Exit code | Meaning |
|-----------|---------|
| 0 | Success |
| 1 | General error (invalid arguments, unexpected failure) |
| 2 | Authentication error (not logged in, token expired, run `oackctl login`) |
| 3 | Authorization error (forbidden, insufficient role) |
| 4 | Resource not found |
| 5 | Validation error (invalid input, 422 response) |

Errors print to stderr. JSON mode also outputs errors as JSON:

```json
{"error": "monitor not found", "code": 4}
```

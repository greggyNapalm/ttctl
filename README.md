# oackctl

CLI for the [Oack](https://oack.io) uptime monitoring API.

## Install

### Homebrew (macOS / Linux)

```bash
brew tap oack-io/tap
brew install oackctl
```

### Shell script

```bash
curl -sSfL "https://raw.githubusercontent.com/oack-io/oackctl/refs/heads/main/install-oackctl.sh" | bash
```

Install a specific version:

```bash
curl -sSfL "https://raw.githubusercontent.com/oack-io/oackctl/refs/heads/main/install-oackctl.sh" | bash -s -- --version 0.3.1
```

Install to a custom directory:

```bash
curl -sSfL "https://raw.githubusercontent.com/oack-io/oackctl/refs/heads/main/install-oackctl.sh" | bash -s -- --dir ~/.local/bin
```

### Manual download

Download the latest release from the [Releases](https://github.com/oack-io/oackctl/releases) page.

| OS | Arch | Download |
|----|------|----------|
| macOS | Apple Silicon | `oackctl_*_darwin_arm64.tar.gz` |
| macOS | Intel | `oackctl_*_darwin_amd64.tar.gz` |
| Linux | x86_64 | `oackctl_*_linux_amd64.tar.gz` |
| Linux | ARM64 | `oackctl_*_linux_arm64.tar.gz` |
| FreeBSD | x86_64 | `oackctl_*_freebsd_amd64.tar.gz` |
| FreeBSD | ARM64 | `oackctl_*_freebsd_arm64.tar.gz` |

## Quick start

```bash
# Authenticate (opens browser for device flow)
oackctl login --server-url https://api.oack.io

# List your teams
oackctl teams list

# List monitors in a team
oackctl monitors list --team <team-id>

# Create a monitor
oackctl monitors create --team <team-id> \
  --name "Production API" \
  --url "https://api.example.com/health" \
  --interval 60

# View probe results
oackctl probes list --team <team-id> --monitor <monitor-id> --limit 10

# Get probe details with CF enrichment, geo, and performance percentiles
oackctl probes details --team <team-id> --monitor <monitor-id> <probe-id>
```

## Command groups

```
oackctl login / logout / whoami / version

oackctl teams          list | get | create | update | delete
oackctl monitors       list | get | create | update | delete | pause | unpause | duplicate | move | test-alert
oackctl probes         list | get | create | update | delete | aggregate | details | pcap
oackctl checkers       list | get | rename | delete | assignments | set-teams | revoke | disconnect | redirect | clear-redirect
oackctl alert-channels list | create | update | delete | test
oackctl monitor-channels list | set | add | remove
oackctl alerts         list
oackctl members        list | add | remove | set-role
oackctl invites        list | create | revoke | accept
oackctl users          list | get | create | update | delete
oackctl preferences    get | set
oackctl geo            regions
oackctl telegram       create-link | link-status

oackctl accounts       list | get | create | update | delete | restore | transfer |
                       members | set-role | remove-member | subscription | update-subscription |
                       create-team | list-teams | create-invite | list-invites | revoke-invite
oackctl status-pages   list | get | create | update | delete |
                       create-component | list-components | update-component | delete-component |
                       create-group | list-groups | update-group | delete-group |
                       create-incident | list-incidents | get-incident | update-incident |
                       delete-incident | post-incident-update |
                       create-maintenance | list-maintenances | get-maintenance |
                       update-maintenance | delete-maintenance | post-maintenance-update |
                       list-subscribers | remove-subscriber |
                       create-watchdog | list-watchdogs | update-watchdog | delete-watchdog |
                       create-template | list-templates | update-template | delete-template
oackctl api-keys       list | create | delete
oackctl comments       create | list | edit | delete | reply | list-replies |
                       resolve | reopen | list-edits | list-by-team | list-by-account
oackctl external-links create | list | get | update | delete | assign | list-by-monitor | unassign
oackctl chart-events   create | list | update | delete
oackctl shares         create | list | revoke
oackctl traces         list | request
oackctl timeline       list
oackctl cf-logs        get | list
oackctl admin          list | get | set-plan | set-status | force-delete | audit-log
oackctl pagerduty      create | get | update | delete | sync
oackctl cf-integrations create | list | get | update | delete
oackctl notifications  defaults | set-defaults | copy-channels |
                       monitor-override | set-monitor-override | remove-monitor-override
oackctl devices        register | list | unregister
```

## Output formats

```bash
# Table output (default)
oackctl teams list

# JSON output
oackctl teams list --json
```

## Configuration

| Flag | Env var | Default | Description |
|------|---------|---------|-------------|
| `--server-url` | `TT_SERVER_URL` | `https://api.oack.io` | API base URL |
| `--token` | `TT_TOKEN` | — | Static Bearer token (skips login) |
| `--profile` | `TT_PROFILE` | `default` | Config profile name |
| `--json` | — | `false` | JSON output |
| `--verbose` | — | `false` | Print HTTP request/response details |

# mw CLI Reference

## Installation

```bash
# macOS (recommended)
brew install mittwald/tap/cli

# npm (any OS, but no auto-update)
npm install -g @mittwald/cli   # requires Node >= 20.7.0

# Docker
docker run --rm -it -e MITTWALD_API_TOKEN mittwald/cli help
```

## Authentication

```bash
mw login token                      # interactive token input
mw login status                     # verify current auth
export MITTWALD_API_TOKEN=<token>   # env var alternative (CI/automation)
```

SSH keypair for app/project ssh/download commands:
```bash
mw user ssh-key create              # generate + import new keypair
mw user ssh-key import              # import existing public key
```

Set SSH identity if multiple keys cause issues:
```bash
export MITTWALD_SSH_IDENTITY_FILE=~/.ssh/mstudio-cli
# or in ~/.ssh/config:
# Host *.project.host
#   IdentityFile ~/.ssh/mstudio-cli
```

## Context System

Avoid repeating `--project-id` etc. by setting a persistent context:

```bash
mw context set --project-id <id>
mw context set --server-id <id>
mw context set --installation-id <id>
mw context get                       # show active context
mw context reset                     # clear context
```

## Global Flags

| Flag | Description |
|---|---|
| `--output json` | JSON output (great for `jq` pipelines) |
| `--output yaml` | YAML output |
| `--quiet` / `-q` | No interactive prompts (CI-safe) |
| `--project-id` / `-p` | Target project |
| `--installation-id` | Target app installation |
| `--server-id` | Target server |
| `--org-id` | Target organization |

## Command Reference

### Projects
```bash
mw project list
mw project get --project-id <id>
mw project create
mw project delete --project-id <id>
mw project ssh                         # SSH into project context
mw project download                    # rsync/SCP files
```

### App Installations
```bash
mw app list                            # available apps in marketplace
mw app install <appname>               # guided install (wordpress, etc.)
mw app get --installation-id <id>
mw app update --installation-id <id>
mw app uninstall --installation-id <id>
mw app ssh                             # SSH into app
mw app download                        # download files
mw app versions --app-id <id>          # available versions
```

### Containers
```bash
mw container list
mw container get --service-id <id>
mw container logs --service-id <id>
mw container restart --service-id <id>
```

### Databases
```bash
mw database list
mw database mysql list
mw database mysql get --database-id <id>
mw database mysql create
mw database mysql delete
mw database mysql dump                 # SSH-based dump
mw database redis list
mw database redis create
```

### Domains & DNS
```bash
mw domain list
mw domain get --domain <domain>
mw dns zone list
mw dns record list
mw dns record set
```

### Users & SSH Keys
```bash
mw user ssh-key list
mw user ssh-key create
mw user ssh-key import
mw user ssh-key delete
```

### Organization / Server
```bash
mw org list
mw server list
mw server get --server-id <id>
```

## Scripting Tips

```bash
# Get all project IDs
mw project list --output json | jq -r '.[].id'

# Get installation ID for a specific app
mw app get --output json | jq '.id'

# Suppress all prompts (CI mode)
mw app install wordpress --quiet --project-id <id> --document-root / --version 6.5

# Check if CLI is authed
mw login status && echo "OK" || echo "NOT AUTHENTICATED"
```

# Marketplace Apps & System Software

## Available Apps (Marketplace)

Check current list:
```bash
mw app list
# or
GET /v2/apps
```

Common apps: `wordpress`, `joomla`, `drupal`, `typo3`, `matomo`, `nextcloud`, `shopware6`

Note: **n8n** is NOT a marketplace app — deploy it as a Docker container (see containers.md).

## Installing an App

```bash
# Guided install
mw app install wordpress

# Non-interactive (CI/automation)
mw app install wordpress \
  --project-id <id> \
  --document-root / \
  --version "6.5.4" \
  --quiet
```

Via API:
```json
POST /v2/projects/{projectId}/appinstallations
{
  "appId": "<wordpress-app-id>",
  "appVersion": { "desired": "<version-id>" },
  "description": "My WordPress",
  "updatePolicy": "patchLevel",
  "documentRoot": "/"
}
```

To get app IDs and version IDs:
```
GET /v2/apps                              # list apps → get appId
GET /v2/apps/{appId}/versions             # list available versions → get versionId
```

## System Software (PHP, Node.js, etc.)

### Finding available software and versions

```
GET /v2/systemsoftwares
GET /v2/systemsoftwares/{softwareId}/versions
```

### Changing PHP version for a WordPress install

1. Find PHP software ID:
```bash
curl -H "Authorization: Bearer $TOKEN" \
  "https://api.mittwald.de/v2/systemsoftwares" | jq '.[] | select(.name=="php")'
```

2. Find target version ID:
```bash
curl -H "Authorization: Bearer $TOKEN" \
  "https://api.mittwald.de/v2/systemsoftwares/<phpId>/versions" | jq '.[] | select(.externalVersion | startswith("8.3"))'
```

3. Apply:
```json
PATCH /v2/appinstallations/{installationId}
{
  "systemSoftware": {
    "<phpSoftwareId>": {
      "systemSoftwareVersion": "<phpVersionId>",
      "updatePolicy": "patchLevel"
    }
  }
}
```

### Via CLI
```bash
mw app get --installation-id <id> --output json   # see current system software
```

## WordPress-Specific Notes

### Redis Object Cache
1. Create Redis database: `POST /v2/projects/{projectId}/redis-databases`
2. Get connection details from response (host, port, password)
3. Configure in `wp-config.php`:
```php
define('WP_REDIS_HOST', '<host>');
define('WP_REDIS_PORT', '<port>');
define('WP_REDIS_PASSWORD', '<password>');
```
4. Install Redis Object Cache plugin

### WordPress Update Policy Options
- `none` — no automatic updates
- `patchLevel` — auto-update patch releases (recommended)
- `all` — auto-update all releases

### SSH into WordPress
```bash
mw app ssh --installation-id <id>
# then: wp cli commands work directly
```

## App Status & Management

```bash
mw app get --installation-id <id>          # details + status
mw app update --installation-id <id>       # trigger update
mw app uninstall --installation-id <id>    # remove app
mw app download --installation-id <id>     # download files via SCP
```

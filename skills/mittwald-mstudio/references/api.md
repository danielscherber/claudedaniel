# mStudio REST API v2 Reference

## Base URL & Spec

- Base URL: `https://api.mittwald.de/v2/`
- OpenAPI spec: `https://api.mittwald.de/v2/openapi.json`
- Docs: `https://developer.mittwald.de/docs/v2/api/intro/`

## Authentication

All requests require:
```
Authorization: Bearer <MITTWALD_API_TOKEN>
```

Create token via UI: *Profil → API-Tokens → Neu erstellen*

Or via API (requires existing token):
```
POST /v2/users/self/api-tokens
{
  "description": "automation",
  "expiresAt": "2026-12-31T23:59:59+00:00",
  "roles": ["api_read", "api_write"]
}
```

Roles: `api_read`, `api_write`

## Key Endpoints by Category

### Projects & Organizations
```
GET    /v2/projects                          # list all projects
GET    /v2/projects?customerId=<orgId>       # filter by org
POST   /v2/projects                          # create project
GET    /v2/projects/{projectId}
DELETE /v2/projects/{projectId}
```

### App Installations
```
GET    /v2/projects/{projectId}/appinstallations
POST   /v2/projects/{projectId}/appinstallations
GET    /v2/appinstallations/{installationId}
PATCH  /v2/appinstallations/{installationId}
DELETE /v2/appinstallations/{installationId}
```

### Container Stacks
```
GET    /v2/projects/{projectId}/stacks       # list stacks (find "default")
GET    /v2/stacks/{stackId}
PUT    /v2/stacks/{stackId}                  # full replace (idempotent!)
PATCH  /v2/stacks/{stackId}                  # partial update

POST   /v2/stacks/{stackId}/services/{serviceId}/actions/start
POST   /v2/stacks/{stackId}/services/{serviceId}/actions/stop
POST   /v2/stacks/{stackId}/services/{serviceId}/actions/restart
GET    /v2/stacks/{stackId}/services/{serviceId}/logs

POST   /v2/projects/{projectId}/registries   # add private registry
PATCH  /v2/registries/{registryId}
DELETE /v2/registries/{registryId}
```

### Databases
```
GET    /v2/projects/{projectId}/mysql-databases
POST   /v2/projects/{projectId}/mysql-databases
GET    /v2/mysql-databases/{databaseId}
DELETE /v2/mysql-databases/{databaseId}

GET    /v2/projects/{projectId}/redis-databases
POST   /v2/projects/{projectId}/redis-databases
GET    /v2/redis-databases/{databaseId}
DELETE /v2/redis-databases/{databaseId}
```

### Domains & Ingresses
```
GET    /v2/projects/{projectId}/domains
POST   /v2/ingresses                         # connect domain to container/app
GET    /v2/ingresses/{ingressId}
PATCH  /v2/ingresses/{ingressId}
DELETE /v2/ingresses/{ingressId}
```

### DNS
```
GET    /v2/projects/{projectId}/dns-zones
GET    /v2/dns-zones/{zoneId}
PATCH  /v2/dns-zones/{zoneId}/record-sets/{recordSet}
```

### System Software (PHP versions etc.)
```
GET    /v2/systemsoftwares                   # list available software
GET    /v2/systemsoftwares/{id}/versions     # list versions for a software
```
To change PHP version for a WordPress install:
1. `GET /v2/systemsoftwares` — find PHP entry, note its ID
2. `GET /v2/systemsoftwares/{phpId}/versions` — find desired version ID
3. `PATCH /v2/appinstallations/{installationId}` with systemSoftware map

## Pagination

Responses with lists support:
- `?limit=<n>` — page size
- `?page=<n>` — page number
- Response headers include `X-Total-Count`

## Error Codes

| Status | Meaning |
|---|---|
| 400 | Bad request — check payload structure |
| 401 | Missing or invalid API token |
| 403 | Insufficient permissions (add `api_write` role) |
| 404 | Resource not found — check IDs |
| 409 | Conflict — resource already exists |
| 422 | Validation error — check field values |

## Example: curl patterns

```bash
# List projects
curl -H "Authorization: Bearer $MITTWALD_API_TOKEN" \
  https://api.mittwald.de/v2/projects

# Start a stopped container service
curl -X POST \
  -H "Authorization: Bearer $MITTWALD_API_TOKEN" \
  -H "Content-Type: application/json" \
  https://api.mittwald.de/v2/stacks/<stackId>/services/<serviceId>/actions/start

# Create a Redis database
curl -X POST \
  -H "Authorization: Bearer $MITTWALD_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"description":"myredis","version":"7.0"}' \
  https://api.mittwald.de/v2/projects/<projectId>/redis-databases
```

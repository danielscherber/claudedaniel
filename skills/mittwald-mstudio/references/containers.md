# Container Stacks on Mittwald mStudio

## Architecture

- Containers run in **stacks** (docker-compose equivalent)
- Each project has exactly one **default** stack
- Multiple services and volumes can live in one stack
- Additional stacks beyond the default are not yet supported

## Finding the Default Stack ID

```bash
# Via CLI
mw container list --output json

# Via API
GET /v2/projects/{projectId}/stacks
# Find the entry with "name": "default" → use its "id"
```

## Full Stack Declaration (PUT)

```json
PUT /v2/stacks/{stackId}

{
  "services": {
    "serviceName": {
      "image": "registry/image:tag",
      "command": ["command", "arg"],
      "entrypoint": ["/entrypoint.sh"],
      "environment": {
        "ENV_VAR": "value"
      },
      "volumes": [
        "volumeName:/container/path",
        "volumeName2:/other/path:ro"
      ],
      "ports": [],
      "depends_on": {
        "otherService": { "condition": "service_healthy" }
      }
    }
  },
  "volumes": {
    "volumeName": {},
    "volumeName2": {}
  }
}
```

⚠️ **PUT is a full replace.** Always include ALL existing services when updating.  
Use `PATCH /v2/stacks/{stackId}` to update individual services without touching others.

## Partial Update (PATCH)

```json
PATCH /v2/stacks/{stackId}

{
  "services": {
    "n8n": {
      "environment": {
        "NEW_VAR": "new_value"
      }
    }
  }
}
```

## n8n Example Stack

```json
{
  "services": {
    "n8n": {
      "image": "n8nio/n8n:latest",
      "environment": {
        "N8N_BASIC_AUTH_ACTIVE": "true",
        "N8N_BASIC_AUTH_USER": "admin",
        "N8N_BASIC_AUTH_PASSWORD": "changeme",
        "WEBHOOK_URL": "https://n8n.your-domain.de/",
        "N8N_HOST": "n8n.your-domain.de",
        "N8N_PROTOCOL": "https"
      },
      "volumes": [
        "n8n_data:/home/node/.n8n"
      ]
    }
  },
  "volumes": {
    "n8n_data": {}
  }
}
```

## Private Registries

Pre-defined (no setup needed):
- `index.docker.io`
- `registry.gitlab.com`
- `ghcr.io`

Add a private registry:
```json
POST /v2/projects/{projectId}/registries
{
  "uri": "my-registry.example.com",
  "description": "My private registry",
  "credentials": {
    "username": "user",
    "password": "secret"
  }
}
```

After adding, use images as: `my-registry.example.com/myimage:tag`

For existing pre-defined registries with private images:
```json
PATCH /v2/registries/{registryId}
{
  "credentials": { "username": "...", "password": "..." }
}
```

## Connecting a Domain

```json
POST /v2/ingresses
{
  "hostname": "n8n.your-domain.de",
  "projectId": "<projectId>",
  "paths": [
    {
      "path": "/",
      "target": {
        "container": {
          "stackId": "<stackId>",
          "serviceName": "n8n",
          "port": 5678
        }
      }
    }
  ]
}
```

## Container Logs

```bash
# Via API
GET /v2/stacks/{stackId}/services/{serviceId}/logs

# Via CLI
mw container logs --service-id <id>
mw container logs --service-id <id> --follow   # tail -f equivalent
```

## Lifecycle Operations

```bash
# Restart a service
POST /v2/stacks/{stackId}/services/{serviceId}/actions/restart

# Stop
POST /v2/stacks/{stackId}/services/{serviceId}/actions/stop

# Start
POST /v2/stacks/{stackId}/services/{serviceId}/actions/start
```

## Common Issues

| Issue | Cause | Fix |
|---|---|---|
| Service disappears after PUT | PUT replaces entire stack | Always include all services in PUT body |
| Image pull fails | Private registry not configured | Add registry via POST /registries |
| Container starts but crashes | Check env vars and volume paths | Inspect logs endpoint |
| Domain returns 502 | Wrong port in ingress | Match port to container's exposed port |
| Container not reachable externally | No ingress defined | Create ingress with POST /v2/ingresses |

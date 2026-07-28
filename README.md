# Plane Config

Plane project management board for the homelab, deployed via Portainer GitOps on nodo1.

## Pattern A: Official Image + Configuration

This repo contains the deployment configuration for Plane. The actual Plane services run from the official `makeplane/plane-*` images published on Docker Hub.

- `docker-compose.yml` — service definitions and configuration
- `.env.example` — template for environment variables (real secrets in `.env`, not in Git)

## Deployment

Deploy via Portainer → Stacks → Repository:

1. URL: `https://github.com/jc2/plane-config`
2. Branch: `main`
3. Compose path: `docker-compose.yml`
4. Enable relative path volumes: ✓ (for bind mount config files)
5. Local filesystem path: `/mnt/portainer-stacks/plane-config`
6. GitOps polling: enabled, interval 5 minutes
7. Environment variables: define in Portainer stack (not in .env)

## Services

| Service | Port | Role |
|---------|------|------|
| backend | 8000 | API server |
| web | 3000 | Frontend (Next.js) |
| db | 5432 | PostgreSQL (internal) |
| redis | 6379 | Cache/sessions (internal) |
| worker | — | Background jobs (internal) |

## First-time setup

1. Copy `.env.example` to `.env` and fill in real values
2. Set `DB_PASSWORD` to a secure password
3. Generate `SECRET_KEY`: `python3 -c "import secrets; print(secrets.token_urlsafe(50))"`
4. Deploy via Portainer

## MCP Testing

Once deployed, test the Plane MCP:

```bash
# From haruhi, with Plane MCP tools loaded
# List all work items in the first project
# Move a work item to a different state
# Create a new issue with description and DoR/DoD
```

See `[[POC de CeronSoft]]` for MCP test plan.

## References

- [Plane GitHub](https://github.com/makeplane/plane)
- [Plane Docs](https://docs.plane.so)
- [Docker Hub: makeplane/plane-backend](https://hub.docker.com/r/makeplane/plane-backend)

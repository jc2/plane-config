# plane-config

Deployment config for [Plane](https://plane.so) on `nodo1`, via Portainer GitOps.
Pattern A (official images + configuration only — nothing is built here).

## What this actually deploys

Plane is a 13-service application, not a web/api/db triple. Leaving any of
these out makes the API return `500` on every route:

| Group | Services | Notes |
|---|---|---|
| Frontends | `web`, `space`, `admin`, `live` | Reached through the proxy, never directly |
| Backend | `api`, `worker`, `beat-worker`, `migrator` | Same image, four different entrypoint scripts |
| Data | `plane-db`, `plane-redis`, `plane-mq`, `plane-minio` | Postgres, Valkey, RabbitMQ, S3 storage |
| Edge | `proxy` | The only service that publishes a port |

`plane-proxy` routes `/`, `/spaces`, `/god-mode`, `/live` and `/api` to the
right service. Hitting the frontend container directly on `:3000` does not work.

## Homelab deviations from upstream

This file tracks the official release compose. Deviations are deliberate and
kept few so an upgrade stays a plain diff against the next release asset:

1. **Image tag pinned** via `APP_RELEASE`. Never `latest` — that tag on Docker
   Hub is roughly two years stale (see "Gotchas").
2. **Proxy binds to `BIND_HOST`**, the node's Tailscale IP, never `0.0.0.0`.
3. **No HTTPS listener.** Tailscale already encrypts node-to-node traffic. TLS
   arrives later with Caddy, when Casdoor needs it.
4. `deploy:` blocks replaced with plain `restart:` (one replica each).
5. Explicit `container_name` on every service.

## Deploying

Portainer → **Stacks → Add stack → Repository**:

| Field | Value |
|---|---|
| Repository URL | `https://github.com/jc2/plane-config` |
| Branch | `main` |
| Compose path | `docker-compose.yml` |
| Enable relative path volumes | off — this stack uses named volumes only |
| GitOps updates | on, 5 minute polling |

Then add the stack environment variables listed in `.env.example`. Every secret
gets its own value:

```bash
python3 -c "import secrets; print(secrets.token_urlsafe(36))"
```

Once healthy, the board is at `http://<BIND_HOST>:<PUBLISHED_PORT>`.

## Gotchas

Worth reading before debugging anything here.

**`:latest` is a trap.** On Docker Hub `makeplane/plane-frontend:latest` and
`plane-backend:latest` were last pushed 2024-10-10. The current release tags are
`v1.3.1` and `stable`. A stack on `latest` silently runs two-year-old code.

**`makeplane/plane-worker` is a dead image.** Last pushed 2023-08-22. The worker
is `plane-backend` running `./bin/docker-entrypoint-worker.sh` — there is no
separate worker image anymore.

**Never override `command`.** The four backend services differ only by their
entrypoint script. Replacing one with a hand-written `gunicorn` invocation skips
the setup those scripts perform, and the API answers `500` on every route.

**Postgres only reads `POSTGRES_PASSWORD` on first init.** Changing the password
in Portainer after the volume exists does nothing, and the API then fails with
`password authentication failed for user "plane"`. To change it, either
`ALTER USER` inside the running container or drop the `pgdata` volume.

**MinIO and RabbitMQ are not optional.** The API needs `AWS_S3_*` for attachments
and `AMQP_URL` for the task queue. Missing either produces the same opaque `500`.

## References

- [Release compose (the source of truth for this file)](https://github.com/makeplane/plane/releases/download/v1.3.1/docker-compose.yml)
- [Plane self-hosting docs](https://developers.plane.so/self-hosting/overview)
- [Plane MCP server](https://developers.plane.so/dev-tools/mcp-server)

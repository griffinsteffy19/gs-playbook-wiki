# Docker Registry

Self-hosted private Docker image registry, used to build and distribute
custom images (starting with [Service Backup](service-backup)) across
all homelab nodes without relying on Docker Hub.

## Where it lives

- **Host:** Synology NAS (`nas.gs`)
- **Deployed via:** Portainer stack (`registry-infra` repo)
- **Storage:** `/volume1/homes/hometeam/registry` (bind-mounted directly,
  no volume driver needed since the registry runs on the NAS itself)
- **Exposed on LAN:** `nas.gs:5050` → proxied via [Traefik](../network-dns/traefik)
  at `https://registry.local.my-domain.com`
- **Source:** private git repo `registry-infra` (GitHub)

## Architecture

Traefik runs on a **separate node** from the registry, so routing uses
Traefik's **file provider** (a static `.yml` route definition) rather
than Docker-label-based discovery, which only works when both containers
share a Docker engine.
[dev machine / node] --https--> [Traefik] --http (LAN)--> [registry:5050 on Synology]

- TLS termination + basic auth: handled by Traefik
- The registry container itself runs plain HTTP internally — never
  exposed directly outside the LAN

## Compose file (`registry-infra/docker-compose.yml`)

```yaml
services:
  registry:
    image: registry:2
    container_name: registry
    restart: unless-stopped
    environment:
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
    volumes:
      - /volume1/homes/hometeam/registry:/data
    ports:
      - "192.168.1.50:5050:5000"   # bound to LAN IP only
```

## Traefik routing

See [Traefik](../network-dns/traefik) for the full dynamic config file. The registry's
specific router/service/middleware block:

```yaml
http:
  routers:
    registry:
      rule: "Host(`registry.local.my-domain.com`)"
      entrypoints:
        - websecure
      tls:
        certResolver: cloudflare
      service: registry
      middlewares:
        - registry-auth

  services:
    registry:
      loadBalancer:
        servers:
          - url: "http://nas.gs:5050"

  middlewares:
    registry-auth:
      basicAuth:
        usersFile: "/etc/traefik/auth/htpasswd"
```

## Auth

Basic auth handled at the Traefik layer via an htpasswd file (not the
registry's own auth mechanism).
- Username: `deploy`
- Password: *(stored securely, not in this wiki)*

To regenerate/add a user:
```bash
docker run --rm -it -v "$(pwd)":/out httpd:2.4-alpine sh -c "htpasswd -Bc /out/htpasswd deploy"
```
Copy the resulting file to the Traefik node at the path referenced in
`usersFile` above.

## Usage

**Log in** (from any dev machine):
```bash
docker login registry.local.my-domain.com
```

**Build and push an image:**
```bash
docker build -t registry.local.my-domain.com/<image-name>:latest .
docker push registry.local.my-domain.com/<image-name>:latest
```

**Pull on a node:**
```bash
docker login registry.local.my-domain.com
docker pull registry.local.my-domain.com/<image-name>:latest
```

**List tags for an image:**
```bash
curl -u deploy:<password> https://registry.local.my-domain.com/v2/<image-name>/tags/list
```

## Verification / health check

```bash
curl -I http://nas.gs:5050/v2/           # direct, bypass Traefik — expect 200 OK
curl -I https://registry.local.my-domain.com/v2/ # through Traefik — expect 401 Unauthorized (auth required, but reachable)
```

## Known gotchas

- Port **5000/5001 conflicts** with DSM's own web UI — registry is
  published on **5050** instead, bound to the LAN IP only.
- **Docker daemon socket permissions on DSM**: commands need `sudo`
  unless your user is in the correct group — check `ls -la /var/run/docker.sock`
  for the owning group, add your user via Control Panel → User & Group.
- `synoservicectl`/`synosystemctl` availability varies by DSM version —
  prefer the Control Panel UI toggle for service restarts.


## Retention / Garbage Collection

Deleting or overwriting a tag (e.g. repeated `docker push` under `:latest`)
does **not** free disk space by itself — old blob data stays on disk until
garbage collection runs and confirms nothing else references it.

### Manual GC

Registry must not be actively receiving pushes during GC (older
`registry:2` GC isn't safe to run concurrently with writes — it can race
and delete a blob mid-push). Stop/start around it rather than trying to
run it live:

```bash
docker stop registry
docker run --rm \
  -e REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/data \
  -v /volume1/homes/hometeam/registry:/data \
  registry:2 \
  garbage-collect /etc/docker/registry/config.yml
docker start registry
```

**The `-e REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/data` env var is
required** on the GC container — without it, GC uses the image's default
config (which expects `/var/lib/registry/...`) instead of matching this
registry's actual storage root (`/data`), and fails with:
failed to garbage collect: failed to mark: filesystem: Path not found: /docker/registry/v2/repositories


### Known gotcha: don't use the `REGISTRY_STORAGE_MAINTENANCE_READONLY_ENABLED` env var

An earlier attempt tried toggling readonly mode via environment variable
instead of a full stop/start, to avoid downtime during GC:
```yaml
environment:
  REGISTRY_STORAGE_MAINTENANCE_READONLY_ENABLED: "true"
```
This **crashes the registry on startup** with:
panic: readonly config key must contain additional keys
`registry:2`'s env-var config mapping can't correctly express the nested
`maintenance.readonly.enabled` structure as a flat env var — it needs an
actual YAML config file to set this properly, which this setup doesn't
use. **Don't use this env var.** The stop/start approach above is the
supported, working method for this deployment.

### Scheduled GC (DSM Task Scheduler)

Control Panel → Task Scheduler → Create → Scheduled Task → User-defined
script. Weekly, off-hours (e.g. 3 AM, after the 2 AM
[Service Backup](service-backup) window):

```bash
#!/bin/bash
docker stop registry
docker run --rm \
  -e REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/data \
  -v /volume1/homes/hometeam/registry:/data \
  registry:2 \
  garbage-collect /etc/docker/registry/config.yml 2>&1 | logger -t registry-gc
docker start registry
```

Same pattern as the earlier decision not to run cron *inside* the
registry or service-backup containers (see service-backup's `supercronic`
gotcha) — DSM's own Task Scheduler runs on the host directly and avoids
container-cron problems entirely.

### Tag-level retention (not yet needed)

Currently only `:latest` is pushed, so there's nothing to prune at the
tag level — GC alone reclaims space from blobs orphaned by repeated
`:latest` overwrites. If versioned tags (`:v1.2.0`, git-sha tags, etc.)
are introduced later, a real retention policy (keep last N tags, delete
older ones via the registry API) will be needed — `registry:2` has no
built-in support for this.
EOF
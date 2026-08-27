# Docker Registry

Self-hosted private Docker image registry, used to build and distribute
custom images (starting with [[Service Backup]]) across all homelab
nodes without relying on Docker Hub.

## Where it lives

- **Host:** Synology NAS (`longsnapper.gs`)
- **Deployed via:** Portainer stack (`registry-infra` repo)
- **Storage:** `/volume1/homes/hometeam/registry` (bind-mounted, no volume driver needed since the registry runs directly on the NAS)
- **Exposed on LAN:** `longsnapper.gs:5050` → proxied via Traefik at
  `https://registry.local.my-domain.com`
- **Source:** private git repo `registry-infra` (GitHub)

## Architecture

Traefik runs on a **separate node** from the registry, so routing uses
Traefik's **file provider** (a static `.yml` route definition) rather
than Docker-label-based discovery, which only works when both containers
share a Docker engine.



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

## Traefik routing (on the Traefik node, not the NAS)

Dynamic config file (file-provider `/routes` directory):
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
          - url: "http://longsnapper.gs:5050"

  middlewares:
    registry-auth:
      basicAuth:
        usersFile: "/etc/traefik/auth/htpasswd"
```

## Auth

Basic auth handled at the Traefik layer via an htpasswd file (not the
registry's own auth mechanism). Credentials:
- Username: `deploy`
- Password: *(stored securely, not in this doc)*

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
curl -I http://longsnapper.gs:5050/v2/           # direct, bypass Traefik — expect 200 OK
curl -I https://registry.local.my-domain.com/v2/ # through Traefik — expect 401 Unauthorized (auth required, but reachable)
```

## Known gotchas

- Port **5000/5001 conflict** with DSM's own web UI — the registry is
  published on **5050** instead, bound to the LAN IP only (not `0.0.0.0`).
- **Docker daemon socket permissions on DSM**: `docker`/`docker compose`
  commands need `sudo` unless your user is in the correct group — check
  `ls -la /var/run/docker.sock` for the owning group and add your user
  via Control Panel → User & Group if needed.
- `synoservicectl`/`synosystemctl` availability varies by DSM version —
  when scripting service restarts, prefer the Control Panel UI toggle
  for reliability across versions.

## To Do / Open Items

- [ ] Set up garbage collection policy for old image layers
- [ ] Decide on retention/versioning strategy (currently only `:latest` is used)
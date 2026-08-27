# Traefik

Reverse proxy handling TLS termination and routing for homelab services,
including the [Docker Registry](../storage-backup/docker-registry). Runs on its own node,
separate from the Synology NAS.

## Where it lives

- **Host:** separate node from the NAS (not `nas.gs`)
- **Certs:** already configured prior to this wiki's setup (existing TLS
  cert resolver in place)

## Why the file provider, not Docker labels

Traefik's Docker provider only discovers containers on the **same Docker
engine** it's attached to. Since services like the registry run on a
different host (the Synology), label-based discovery doesn't work across
hosts — routes for those services are defined manually instead, via
Traefik's **file provider**.

## Static config

```yaml
providers:
  docker:
    exposedByDefault: false
  file:
    directory: /routes
    watch: true
```

Dynamic route files live in `/routes` inside the Traefik container
(`watch: true` means new/changed files are picked up live, no restart
needed).

## Adding a new route (for a service on another host)

Create a new `.yml` file in `/routes` following this pattern (this is the
registry's actual route, as a template):

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

Key values to reuse/confirm for new routes:
- **`entrypoints`**: `websecure` (confirmed working — verify against
  `traefik.yml`'s `entryPoints` block if unsure, names must match exactly)
- **`certResolver`**: `cloudflare` (confirmed working)
- **`usersFile`** path: `/etc/traefik/auth/htpasswd` — must be mounted
  into the Traefik container as a volume; same file can be reused across
  multiple routers' basic-auth middlewares

## Basic auth (htpasswd)

Used to protect routes like the registry. Generate/update:
```bash
docker run --rm -it -v "$(pwd)":/out httpd:2.4-alpine sh -c "htpasswd -Bc /out/htpasswd deploy"
```
Copy the resulting file to the Traefik node at the mounted path.

## Verifying a new route

```bash
curl -I http://<backend-host>:<port>/           # direct to backend, bypass Traefik
curl -I https://<hostname>/                     # through Traefik
```
A `401` through Traefik (when auth middleware is attached) confirms
routing + TLS + auth are all correctly wired — it means Traefik matched
the router and proxied through, just refused for lack of credentials.
A `404` through Traefik instead means **no router matched** — check for
YAML indentation errors (Traefik silently skips malformed dynamic config
files rather than erroring loudly) or an entrypoint/hostname mismatch.

## Known gotchas

- **YAML indentation errors fail silently.** Traefik logs a parse error
  but doesn't crash — the route just doesn't exist. Always check
  `docker logs traefik` after adding/editing a route file, and verify
  with `docker exec traefik cat /routes/<file>.yml` that what's on disk
  matches what you intended.
- **Entrypoint/resolver names are instance-specific.** `websecure` and
  `cloudflare` are this Traefik instance's actual configured names —
  don't assume they're universal defaults; confirm via
  `docker exec traefik cat /etc/traefik/traefik.yml` if setting up a
  new instance elsewhere.
- **Traefik dashboard** (if enabled) is the fastest way to confirm a
  router loaded correctly — check HTTP → Routers for a green/error state
  before troubleshooting further via curl.

## To Do / Open Items

- [ ] Document full static `traefik.yml` (entrypoints, cert resolver
      definition, dashboard access)
- [ ] Document which other services already route through this Traefik
      instance

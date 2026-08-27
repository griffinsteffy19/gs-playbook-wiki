# Homelab

Central index for infrastructure, services, and how things are configured.
This page is the starting point — add new sections/links here as new
services or systems come online.

## Infrastructure Overview

| Component        | Host             | Notes                                  |
|-------------------|------------------|-----------------------------------------|
| Docker Registry   | Synology NAS (`longsnapper.gs`) | Private image registry for self-built containers |
| Reverse Proxy     | Traefik (separate node) | TLS termination, routing via file provider |
| Backup Target     | Synology NAS (`longsnapper.gs`) | rsync daemon, module `backup` |
| Node: holder      | *(fill in host details)* | Runs media/service stack + service-backup container |

## Services & Documentation

- [[Docker Registry]] — self-hosted private image registry, setup and usage
- [[Service Backup]] — containerized config backup system, deployment and troubleshooting
- *(add more service pages here as they're documented)*

## Nodes

### holder
- Runs: gluetun, prowlarr, radarr, sonarr, tautulli, kometa, seerr
- Services live under: `/opt/data/<service>`
- Backed up via: [[Service Backup]]

*(add additional nodes here as they're brought into the backup system / documented)*

## Conventions

- Service configs live at `/opt/data/<service>` on each node.
- Backups land on the NAS at `/volume1/homes/hometeam/backups/<node-name>/<service>/<timestamp>/`.
- Private repos: `service-backup`, `registry-infra` (git host: GitHub, private).
- Registry: `registry.local.my-domain.com`, routed through Traefik.

## To Do / Open Items

- [ ] Document Traefik setup itself (routing conventions, cert resolver name, dynamic config location)
- [ ] Add remaining nodes to service-backup rollout
- [ ] Consider garbage collection policy for the registry
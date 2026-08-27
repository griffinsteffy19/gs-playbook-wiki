# Homelab

Central index for infrastructure, services, and how things are configured.

This is the starting point — expand this page as new services, nodes, or
systems come online.

## Infrastructure Overview

| Component        | Host                          | Notes |
|-------------------|-------------------------------|-------|
| Synology NAS      | `nas.domain`               | Hosts Docker Registry, rsync backup target, Wikmd |
| Docker Registry   | `nas.domain:5050` (LAN) → `registry.local.my-domain.com` | Private image registry, proxied via Traefik |
| Reverse Proxy     | Traefik (separate node from NAS) | TLS termination, routing via file provider |
| Backup Target     | `nas.domain`, rsync daemon | Module `backup` → `/volume1/homes/hometeam/backups/` |
| Wiki              | `nas.domain` via Wikmd     | This wiki — Docker container, port 9454 |
| Node: holder      | **(fill in: IP/hostname)**       | Runs media/service stack, backed up via service-backup container |
| Proxmox Cluster   | `Hometeam`                  | Multi-node virtualization cluster |
| Router / Firewall | Ubiquiti UDM Pro            | Routing, VLANs, firewall policy, UniFi network management |
| DNS               | Pi-hole                     | Internal DNS and local wildcard records |
| Home Automation   | Home Assistant OS VM        | Runs on Proxmox |

## Services & Documentation

- [Docker Registry](docker-registry) — self-hosted private image registry
- [Service Backup](service-backup) — containerized config backup system
- [Traefik](traefik) — reverse proxy / TLS / routing
- [Wiki (Wikmd)](wikimd) — this wiki's own setup, for future reference

## Additional Homelab Documentation

These pages are broader infrastructure references and are intended to complement,
not replace, the service-specific pages above.

- [Proxmox Cluster](proxmox-cluster) — cluster, storage, VM, and thin-pool notes
- [Networking & DNS](networking-dns) — UDM Pro, Pi-hole, VLAN, and internal DNS notes
- [Storage & NAS](storage-nas) — Synology, CIFS mounts, local vs NAS storage
- [Backup Strategy](backup-strategy) — VM backups, service backups, and restore priorities
- [Media Stack](media-stack-overview) — holder media services and VPN relationships
- [Home Assistant Infrastructure](home-assistant-infra) — HA VM and infrastructure integrations
- [Monitoring](monitoring-overview) — Proxmox, UniFi, Glances, NAS, and HA monitoring
- [Incident Notes](incident-notes) — important outages, disk incidents, and lessons learned

## Nodes

### holder

- Runs: `backup`, `gluetun`, `prowlarr`, `radarr`, `sonarr`, `tautulli`, `kometa`, `seerr`
- Service configs live at: `/opt/data/<service>`
- Backed up nightly (2 AM) via [Service Backup](service-backup)
- Note: `overseerr` was renamed to `seerr` — the service-backup config
  was updated to match (an `overseerr.bk` folder exists from before the
  rename and is intentionally not backed up)

**(add additional nodes here as they're brought into the backup rollout)**

### Other Known Infrastructure Names

The following names have been referenced in the Proxmox / VM environment.
This is intentionally an inventory aid only; add IPs, VM IDs, and exact roles
here only once confirmed.

- `gm`
- `st-coordinator`
- `o-coordinator`
- `fullback`
- `cold-plunge`
- `blue-tent`
- `punter`
- `kidney`
- `record-book`

See [Proxmox Cluster](proxmox-cluster) for currently documented details.

## Git Repositories (private, GitHub)

| Repo | Contents |
|---|---|
| `service-backup` | Backup container source (Dockerfile, backup.sh, entrypoint.sh, compose file) |
| `registry-infra` | Docker registry's compose file |

## Conventions

- Service configs live at `/opt/data/<service>` on each node.
- Backups land on the NAS at
  `/volume1/homes/hometeam/backups/<node-name>/<service>/<timestamp>/`,
  as dated snapshots with hard-link-based retention (not flat mirrors).
- Registry images: `registry.local.my-domain.com/<image>:latest`.
- Wiki pages: plain Markdown links to filename slugs, e.g.
  `[Display Text](page-slug)` — Wikmd does **not** use `[[wikilink]]` syntax.
- DSM quirks to remember: `synoservicectl`/`synosystemctl` availability
  varies by DSM version — prefer the Control Panel UI toggle for
  restarting services when a CLI command doesn't exist. Docker commands
  on the NAS often need `sudo` unless your user's in the right group.
- The wiki documents configuration and operational knowledge, but should
  not contain live passwords, API tokens, private keys, or `.env` secret values.

## To Do / Open Items

- [ ] Fill in `holder`'s IP/hostname above
- [ ] Add remaining nodes to service-backup rollout
- [ ] Decide on registry garbage collection policy
- [ ] Confirm `.env` credentials/secrets are stored somewhere safe outside
      the wiki (this wiki documents *how* things are configured, not
      live passwords)
- [ ] Add confirmed Proxmox node IPs and primary roles
- [ ] Add VM ID → hostname → purpose → backup-policy inventory
- [ ] Document VLANs and inter-VLAN firewall rules
- [ ] Add storage-capacity / thin-pool monitoring thresholds

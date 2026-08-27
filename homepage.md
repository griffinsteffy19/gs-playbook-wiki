# Homelab

Central index for infrastructure, services, and how things are configured.
This is the starting point — expand this page as new services, nodes, or
systems come online.

## Infrastructure Overview

| Component        | Host                          | Notes |
|-------------------|-------------------------------|-------|
| Synology NAS      | `nas.gs`               | Hosts Docker Registry, rsync backup target, Wikmd |
| Docker Registry   | `nas.gs:5050` (LAN) → `registry.local.<domain>` | Private image registry, proxied via Traefik |
| Reverse Proxy     | Traefik (separate node from NAS) | TLS termination, routing via file provider |
| Backup Target     | `nas.gs`, rsync daemon | Module `backup` → `/volume1/homes/hometeam/backups/` |
| Wiki              | `nas.gs` via Wikmd     | This wiki — Docker container, port 9454 |
| Node: holder      | **(fill in: IP/hostname)**       | Runs media/service stack, backed up via service-backup container |
| Proxmox Cluster   | `Hometeam`                  | Multi-node virtualization cluster — see [Host and VM Inventory](core-infrastructure/host-and-vm-inventory) |
| Router / Firewall | Ubiquiti UDM Pro            | Routing, VLANs, firewall policy, UniFi network management |
| DNS               | Pi-hole                     | Internal DNS and local wildcard records |
| Home Automation   | Home Assistant OS VM        | Runs on Proxmox |

## Documentation Index

### Core Infrastructure
- [Homelab Architecture](core-infrastructure/homelab-architecture)
- [Hardware Inventory](core-infrastructure/hardware-inventory)
- [Host and VM Inventory](core-infrastructure/host-and-vm-inventory) — canonical node/VM table
- [Proxmox Cluster](core-infrastructure/proxmox-cluster)
- [Proxmox Storage Runbook](core-infrastructure/proxmox-storage-runbook)
- [VM Disk, Discard, and TRIM](core-infrastructure/vm-disk-and-trim) — canonical thin-provisioning reference
- [Wake-on-LAN and Power Management](core-infrastructure/wake-on-lan-and-power)
- [gm Proxmox Node](core-infrastructure/gm-proxmox-node)

### Network & DNS
- [Networking & DNS](network-dns/networking-dns)
- [DNS and Name Resolution](network-dns/dns-and-name-resolution)
- [UniFi Firewall and VLANs](network-dns/unifi-firewall-and-vlans) — canonical cross-VLAN rules
- [Security Boundaries](network-dns/security-boundaries)
- [Network and Service Dependency Map](network-dns/network-service-dependency-map)
- [Reverse Proxy and Access Paths](network-dns/reverse-proxy-access-paths)
- [Traefik](network-dns/traefik)
- [Pi-hole Operations](network-dns/pihole-operations)
- [Authelia Operations](network-dns/authelia-operations)

### Storage & Backup
- [Synology Storage Architecture](storage-backup/synology-storage-architecture) — canonical NAS reference
- [CIFS / SMB Mount Standards](storage-backup/cifs-smb-mount-standards)
- [Backup Strategy](storage-backup/backup-strategy)
- [Backup and Restore Matrix](storage-backup/backup-restore-matrix)
- [Docker Backup and Restore Runbook](storage-backup/docker-backup-restore-runbook)
- [Docker Data Directory Standard](storage-backup/docker-data-directory-standard)
- [Docker Registry](storage-backup/docker-registry)
- [Service Backup](storage-backup/service-backup)

### Services
- [Docker Hosts and Service Layout](services/docker-hosts-and-service-layout)
- [Media Stack Overview](services/media-stack-overview)
- [Media Networking and VPN](services/media-networking-and-vpn) — canonical Gluetun/VPN reference
- [Plex and Tautulli Operations](services/plex-and-tautulli-operations)
- [Jellyfin Operations](services/jellyfin-operations)
- [Kometa Operations](services/kometa-operations)
- [Immich Operations and Storage](services/immich-operations-and-storage)
- [Vaultwarden Operations](services/vaultwarden-operations)
- [Wiki (Wikmd)](services/wikimd)

### Home Assistant
- [Home Assistant Infrastructure](home-assistant/home-assistant-infra)
- [Home Assistant Homelab Monitoring](home-assistant/home-assistant-homelab-monitoring)
- [Home Assistant Backup and Recovery](home-assistant/home-assistant-backup-and-recovery)
- [Home Assistant Dynamic Temperature](home-assistant/home-assistant-dynamic-temperature)
- [HVAC Runtime Tracking](home-assistant/hvac-runtime-tracking)
- [Home Assistant Internet Resilience](home-assistant/home-assistant-internet-resilience)

### Operations & Incident Response
- [Disaster Recovery Runbook](operations/disaster-recovery-runbook)
- [Incident Notes](operations/incident-notes)
- [Troubleshooting Cheatsheet](operations/troubleshooting-cheatsheet)
- [Maintenance Checklists](operations/maintenance-checklists)
- [Monitoring Overview](operations/monitoring-overview)
- [Glances Monitoring](operations/glances-monitoring)
- [Ubuntu Host Baseline](operations/ubuntu-host-baseline)

## Nodes

### holder

- Runs: `backup`, `gluetun`, `qbittorrent`, `prowlarr`, `radarr`, `sonarr`, `tautulli`, `kometa`, `seerr`
- Service configs live at: `/opt/data/<service>`
- Backed up nightly (2 AM) via [Service Backup](storage-backup/service-backup)
- Note: `overseerr` was renamed to `seerr` — the service-backup config
  was updated to match (an `overseerr.bk` folder exists from before the
  rename and is intentionally not backed up)

**(add additional nodes here as they're brought into the backup rollout)**

Full node/VM inventory: [Host and VM Inventory](core-infrastructure/host-and-vm-inventory).

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
- Registry images: `registry.local.<domain>/<image>:latest`.
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

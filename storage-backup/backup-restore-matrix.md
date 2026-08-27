# Backup and Restore Matrix

## Purpose

This page turns backup philosophy into a concrete service-by-service recovery model.

## Suggested Matrix

| System | Configuration | Database | Primary Data | VM Backup | Service Backup | Regenerable Data | Restore Priority |
|---|---|---|---|---|---|---|---|
| Home Assistant | Yes | Internal | HA state | Yes | HA-native | caches | Critical |
| Vaultwarden | Yes | Yes | vault data | Prefer | Yes | minimal | Critical |
| Immich | Yes | Yes | originals | Selective | Yes | thumbs/encoded video | High |
| Traefik | Yes | n/a | cert state | Optional | Yes | none | High |
| Authelia | Yes | Depends | identity config | Optional | Yes | none | High |
| Pi-hole | Yes | n/a | DNS config | Optional | Yes | query logs | High |
| Sonarr | Yes | Yes | metadata | Optional | Yes | cache | Medium |
| Radarr | Yes | Yes | metadata | Optional | Yes | cache | Medium |
| Prowlarr | Yes | Yes | indexer config | Optional | Yes | cache | Medium |
| qBittorrent | Yes | small state | incomplete downloads | Usually no | Selective | downloads | Low/Medium |
| Tautulli | Yes | Yes | history | Optional | Yes | cache | Medium |
| Kometa | Yes | n/a | config | No | Yes | generated metadata | Medium |
| Seerr | Yes | Yes | request history | Optional | Yes | cache | Medium |
| WikiMD | Yes | file-based | wiki pages | Optional | Yes | none | Medium |
| Registry | Yes | metadata | images | Optional | Depends | images reproducible from CI | Medium |

## Recovery Priority Tiers

### Tier 0 — foundational

Must recover first:

- network
- DNS
- NAS
- Proxmox
- reverse proxy

### Tier 1 — critical state

Then recover:

- Home Assistant
- Vaultwarden
- authentication
- critical databases

### Tier 2 — user-facing applications

Then:

- Immich
- Plex / media apps
- WikiMD
- request / automation services

### Tier 3 — regenerable infrastructure

Finally:

- image registry content
- caches
- thumbnails
- encoded media

## Restore Testing

Record:

| System | Last Restore Test | Result | Notes |
|---|---|---|---|
| Home Assistant | | | |
| Vaultwarden | | | |
| Immich | | | |
| service-backup snapshot | | | |
| Proxmox VM | | | |

## Questions to Resolve

- Which VM backups are stored only on the NAS?
- Which backups exist offsite?
- Which services depend on secrets stored only in one password manager?
- Can the NAS itself be restored if it fails?
- Is the wiki available during a NAS outage?

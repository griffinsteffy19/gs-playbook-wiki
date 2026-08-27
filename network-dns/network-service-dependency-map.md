# Network and Service Dependency Map

## Purpose

This page documents which infrastructure services depend on which other systems.

## Core Chain

```text
UDM Pro
   |
   +--> Pi-hole
   |      |
   |      +--> internal service DNS
   |
   +--> Traefik
          |
          +--> Authelia
          |
          +--> application backends
```

## Storage Dependencies

```text
Synology NAS
   |
   +--> media stack
   +--> Plex / Jellyfin
   +--> Immich originals
   +--> service-backup target
   +--> Docker registry data
   +--> WikiMD
```

The NAS is therefore a high-impact dependency.

## Media Stack Dependency Chain

```text
UDM Pro / LAN
   |
   v
holder
   |
   +--> Gluetun
   |      |
   |      +--> qBittorrent
   |      +--> Sonarr
   |      +--> Radarr
   |      +--> Prowlarr
   |
   +--> Tautulli
   +--> Kometa
   +--> Seerr
   |
   v
Synology media shares
```

## Reverse Proxy Dependency Chain

```text
client
  |
  v
Pi-hole / public DNS
  |
  v
Traefik
  |
  +--> Authelia (selected services)
  |
  v
backend application
```

## Home Assistant Dependency Chain

```text
Proxmox
   |
   v
Home Assistant OS VM
   |
   +--> UniFi
   +--> Synology
   +--> Proxmox API
   +--> Plex
   +--> Tautulli
   +--> Ambient Weather
   +--> local device VLANs
```

## Backup Dependency Chain

```text
Docker host
   |
   v
service-backup
   |
   v
network
   |
   v
Synology rsync module
   |
   v
/volume1/homes/hometeam/backups/
```

## Failure Impact Examples

### Pi-hole down

Possible symptoms:

- internal service names stop resolving
- HTTPS services appear unavailable by hostname
- direct IP access may still work

### Traefik down

Possible symptoms:

- proxied services unavailable
- direct host:port access may still work

### Synology down

Possible symptoms:

- media unavailable
- backups fail
- WikiMD unavailable
- registry may fail
- NAS-backed mounts disappear

### Gluetun down

Possible symptoms:

- qBittorrent loses external connectivity
- Sonarr/Radarr/Prowlarr may lose internet/indexer access

### Proxmox node down

Impact depends on VM placement and HA configuration.

## Recommendation

Use this dependency map when deciding:

- startup order
- monitoring priority
- backup priority
- maintenance windows
- incident response order

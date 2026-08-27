# Media Networking and VPN

## Overview

The media automation stack uses Gluetun to provide VPN egress through Private Internet Access.

## Known Services

Primary media automation host:

```text
holder
```

Known services:

- qBittorrent
- Sonarr
- Radarr
- Prowlarr
- Tautulli
- Kometa
- Seerr
- backup service
- Gluetun

## Gluetun Pattern

Selected services use:

```yaml
network_mode: service:gluetun
```

This means those containers share Gluetun's network namespace.

## Consequence

If Gluetun is unhealthy, several downstream services may appear to have networking problems
at the same time.

Troubleshooting order:

1. Gluetun container state
2. VPN authentication
3. VPN tunnel establishment
4. DNS from inside Gluetun
5. outbound connectivity
6. dependent service health

## qBittorrent

qBittorrent should be treated as directly dependent on the VPN layer.

Confirm:

- container is attached through Gluetun
- expected listening port behavior
- downloads map to correct storage
- Sonarr/Radarr paths match qBittorrent paths

## Sonarr / Radarr / Prowlarr

If all indexers fail simultaneously, check the VPN path before editing indexer configuration.

## Media Storage

Bulk media resides on the Synology NAS.

Typical Linux-side mount paths:

```text
/data/movies
/data/shows
```

Ensure Docker path mappings are consistent between:

- qBittorrent
- Sonarr
- Radarr
- Plex / Jellyfin

## Path Mapping Principle

Prefer the same in-container path for shared media/download locations where possible.

Example conceptual layout:

```text
/data/downloads
/data/movies
/data/shows
```

This avoids remote-path mapping complexity.

## Failure Modes

### Sonarr/Radarr cannot import completed download

Check:

- container bind mounts
- in-container paths
- file ownership
- permissions
- qBittorrent category path
- NAS mount availability

### Indexers fail but LAN UI works

Check:

- Gluetun
- PIA tunnel
- DNS inside VPN namespace

### Plex/Jellyfin sees missing media

Check:

- NAS mount
- mount persistence
- UID/GID permissions
- container bind mount

# Plex and Tautulli Operations

## Overview

Plex is the primary media server.

Tautulli provides Plex activity and historical monitoring.

Media is stored primarily on the Synology NAS.

## Storage

Typical media mount paths:

```text
/data/movies
/data/shows
```

Before troubleshooting Plex itself, verify NAS mounts:

```bash
mountpoint /data/movies
mountpoint /data/shows
```

## Hardware Transcoding

AMD integrated graphics passthrough was explored.

An AMD 680M path was tested but Proxmox/QEMU passthrough was not stable enough to treat as
production architecture.

A `vainfo` error was also encountered in Linux when no suitable display / acceleration
context was available.

Treat hardware transcoding as optional until a known-stable device path exists.

## Tautulli

Tautulli is used for:

- current stream visibility
- history
- bandwidth
- transcode monitoring
- Home Assistant integration

## Common Troubleshooting

### Plex libraries empty

Check:

- NAS mount
- container bind mount
- file permissions
- Plex library path

### Playback fails

Check:

- file reachable
- codec
- transcode requirement
- CPU load
- hardware acceleration
- network bandwidth

### Tautulli shows no data

Check:

- Plex API connectivity
- token
- container state
- Tautulli logs

## Backup Scope

Back up:

- Plex metadata/config if preserving watch state/library customization matters
- Tautulli database/config
- Compose files

Media itself is backed up / protected separately through NAS strategy.

## Home Assistant

Useful top-level entities include:

- Plex availability
- active streams
- transcodes
- Tautulli session count

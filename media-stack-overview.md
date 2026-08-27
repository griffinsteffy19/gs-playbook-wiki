# Media Stack Overview

## Host

Primary media / automation host:

```text
holder
```

Known services include:

- `backup`
- `gluetun`
- `qbittorrent`
- `prowlarr`
- `radarr`
- `sonarr`
- `tautulli`
- `kometa`
- `seerr`

Configuration convention:

```text
/opt/data/<service>
```

## VPN Relationship

Gluetun provides VPN networking for selected media automation services.

Typical Docker pattern:

```yaml
network_mode: service:gluetun
```

Known services that have used this path:

- qBittorrent
- Sonarr
- Radarr
- Prowlarr

When several of these services lose internet connectivity at once, check Gluetun
before troubleshooting each application individually.

## Plex / Tautulli

Plex media is stored on the Synology NAS.

Tautulli is used for Plex monitoring.

Hardware transcoding has been investigated using integrated AMD graphics, but GPU
passthrough should be treated as optional unless the host / guest path is known stable.

## Jellyfin

Jellyfin has also been deployed behind Traefik.

Collection/tag propagation has been used to support organization and parental-control
workflows.

## Kometa

Known configuration path:

```text
/opt/data/kometa/config
```

Collections / automation previously worked on include:

- James Bond
- Star Wars
- MCU
- Pixar
- Disney

Radarr preference previously used:

```yaml
add_missing: true
search: false
```

## Seerr Rename

`overseerr` was renamed to:

```text
seerr
```

The service-backup configuration was updated accordingly.

An old:

```text
overseerr.bk
```

directory may still exist and is intentionally not part of active backups.

## To Add

- [ ] Compose-file locations
- [ ] service ports
- [ ] Traefik routes
- [ ] NAS mount mapping
- [ ] restart order / dependencies
- [ ] media-library paths

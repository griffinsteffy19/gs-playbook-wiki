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

Configuration convention: `/opt/data/<service>` — see
[Docker Data Directory Standard](../storage-backup/docker-data-directory-standard).

## VPN Relationship

Gluetun provides VPN networking for `qbittorrent`, `sonarr`, `radarr`, and
`prowlarr`. See [Media Networking and VPN](media-networking-and-vpn) for
the full Gluetun architecture, troubleshooting order, and failure modes.

## Plex / Tautulli

Plex media is stored on the Synology NAS.

Tautulli is used for Plex monitoring. See
[Plex and Tautulli Operations](plex-and-tautulli-operations) for
hardware transcoding notes and troubleshooting.

## Jellyfin

Jellyfin has also been deployed behind Traefik. Collection/tag propagation
has been used to support organization and parental-control workflows.
See [Jellyfin Operations](jellyfin-operations).

## Kometa

Known configuration path:

```text
/opt/data/kometa/config
```

Collections / automation previously worked on include: James Bond, Star
Wars, MCU, Pixar, Disney.

Radarr preference previously used:

```yaml
add_missing: true
search: false
```

See [Kometa Operations](kometa-operations) for full details.

## Seerr Rename

`overseerr` was renamed to `seerr`. The service-backup configuration was
updated accordingly. An old `overseerr.bk` directory may still exist and
is intentionally not part of active backups.

## To Add

- [ ] Compose-file locations
- [ ] service ports
- [ ] Traefik routes
- [ ] NAS mount mapping
- [ ] restart order / dependencies
- [ ] media-library paths

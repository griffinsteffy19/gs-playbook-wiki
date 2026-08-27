# Docker Hosts and Service Layout

## Convention

Docker application data follows the `/opt/data/<service>` convention — see
[Docker Data Directory Standard](../storage-backup/docker-data-directory-standard) for the
full rationale, suggested layout, and permissions/migration guidance.

## `holder`

Primary media automation host.

Known services:

```text
backup
gluetun
qbittorrent
prowlarr
radarr
sonarr
tautulli
kometa
seerr
```

### Service Backup

`holder` is backed up nightly at approximately `02:00` using the
service-backup container.

The old `overseerr.bk` folder remains from before the `overseerr` ->
`seerr` rename and is intentionally not included in the active backup set.

## `record-book`

Known application / Docker host.

Previously associated services include:

- Immich
- Vaultwarden
- Portainer CE

### Storage Pattern

```text
/     -> operating system and application binaries
/opt/data -> application configuration / state
/data -> larger local working data
NAS mounts -> shared / bulk storage
```

## Docker Log Rotation

```yaml
logging:
  driver: json-file
  options:
    max-size: "10m"
    max-file: "3"
```

This prevents container logs from growing without bound.

## Common Operational Commands

```bash
docker ps
docker ps -a
docker compose ps
docker logs <container>
docker logs -f <container>
docker system df
docker system df -v
docker inspect <container>
docker compose restart <service>
docker compose up -d
docker compose pull
```

## Migration Checklist

When moving a service between Docker hosts:

1. identify Compose file
2. identify `/opt/data/<service>`
3. identify secrets / `.env`
4. identify external networks
5. identify Traefik labels
6. identify bind mounts
7. identify NAS mounts
8. identify VPN dependencies
9. stop old container
10. copy application state
11. start on new host
12. test internally
13. test through reverse proxy
14. verify backup coverage
15. remove old instance only after validation

## Data Classification

### Usually important

- databases
- service configuration
- user-uploaded data
- custom scripts
- authentication state

### Usually regenerable

- container images
- caches
- thumbnails
- transcodes
- temporary downloads

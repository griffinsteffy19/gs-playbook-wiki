# Backup Strategy

## Existing Backup Layers

The homelab currently uses more than one backup mechanism.

### Proxmox VM Backups

Used for whole-VM recovery where appropriate.

Home Assistant already has regular backup coverage.

### Service-Level Backups

The existing [Service Backup](service-backup) system backs up application configuration
and service data.

Known convention:

```text
/opt/data/<service>
```

Known destination pattern:

```text
/volume1/homes/hometeam/backups/<node-name>/<service>/<timestamp>/
```

The service-backup system uses dated snapshots with hard-link-based retention.

### Synology NAS

The NAS is the central destination for several backup workflows.

## Backup Priority

### Highest priority

- Home Assistant configuration / backups
- Vaultwarden
- databases
- reverse-proxy configuration
- authentication configuration
- Docker Compose / application configuration
- custom scripts
- irreplaceable user data

### Lower priority / regenerable

- Docker images
- package caches
- thumbnails
- transcoded / encoded media
- other generated application artifacts

## Restore Principle

A backup is not considered complete until restore has been tested.

Recommended periodic tests:

1. restore a service configuration
2. restore a database
3. restore a VM
4. verify required secrets are available
5. verify DNS / reverse-proxy dependencies are documented

## Secrets

The wiki should document:

- where a secret is stored
- which service uses it
- how that service consumes it

Do not place the actual secret value in WikiMD.

## Existing Service-Backup Notes

`holder` is backed up nightly at 2 AM using the service-backup container.

The old:

```text
overseerr.bk
```

directory predates the rename to `seerr` and is intentionally excluded from the
current backup configuration.

## To Add

- [ ] backup schedule by VM
- [ ] backup schedule by service
- [ ] retention policy
- [ ] restore test date / result
- [ ] off-NAS / offsite backup coverage
- [ ] secret-storage location

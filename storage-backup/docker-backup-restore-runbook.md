# Docker Backup and Restore Runbook

## Purpose

Operational procedure for restoring a Docker service from the homelab service-backup system.

## Assumptions

Persistent service state normally lives at:

```text
/opt/data/<service>
```

Backups land under:

```text
/volume1/homes/hometeam/backups/<node>/<service>/<timestamp>/
```

## Before Restore

Identify:

- service name
- target host
- Compose file
- backup timestamp
- `.env` / secrets
- NAS mounts
- Traefik route
- database dependency
- VPN dependency

## Stop Service

```bash
docker compose stop <service>
```

or stop the entire stack if service consistency requires it:

```bash
docker compose down
```

## Preserve Current State

Move or snapshot the current data:

```bash
mv /opt/data/<service> /opt/data/<service>.pre-restore
```

Do not immediately delete the failed state.

## Restore Backup

Copy the selected snapshot back into place.

Use `rsync` if appropriate:

```bash
rsync -aHAX <backup>/ /opt/data/<service>/
```

## Permissions

Verify:

```bash
ls -la /opt/data/<service>
stat /opt/data/<service>
```

Correct UID/GID if required.

## Start

```bash
docker compose up -d
```

## Validate

Check:

```bash
docker compose ps
docker compose logs
```

Then validate:

1. local backend
2. reverse-proxy URL
3. login
4. database state
5. NAS mounts
6. scheduled tasks
7. backup job

## Roll Back Restore

If the selected backup is bad:

```bash
docker compose down
rm -rf /opt/data/<service>
mv /opt/data/<service>.pre-restore /opt/data/<service>
docker compose up -d
```

Only do this after verifying the preserved directory is intact.

## Post-Restore

Document:

- date
- backup used
- reason
- success/failure
- any missing secrets
- any schema migration issues

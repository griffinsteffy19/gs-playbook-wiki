# Docker Data Directory Standard

## Standard

Application data lives under:

```text
/opt/data/<service>
```

This convention is one of the most useful structural standards in the homelab.

## Why

It provides:

- predictable backup paths
- easier migration
- simpler permissions
- clearer disaster recovery
- easier inspection of storage usage

## Suggested Layout

```text
/opt/data/
├── traefik/
├── authelia/
├── vaultwarden/
├── sonarr/
├── radarr/
├── prowlarr/
├── tautulli/
├── kometa/
├── seerr/
└── <service>/
```

Each service folder should contain only persistent data needed for that application.

## Compose Files

Keep the Compose file location documented.

Two acceptable models:

```text
/opt/data/<service>/docker-compose.yml
```

or:

```text
/opt/stacks/<service>/docker-compose.yml
/opt/data/<service>/...
```

Consistency matters more than which model is chosen.

## Secrets

Prefer:

```text
.env
```

or dedicated secret files outside the wiki.

Never commit secrets to Git.

## Backup Classification

For each directory, classify as:

```text
critical
important
regenerable
temporary
```

## Permissions

Record:

- UID
- GID
- owner
- group
- expected mode

Example inspection:

```bash
ls -ld /opt/data/<service>
stat /opt/data/<service>
```

## Migration Procedure

```bash
docker compose stop
rsync -aHAX /opt/data/<service>/ <new-host>:/opt/data/<service>/
docker compose up -d
```

Verify:

- ownership
- secrets
- networks
- bind mounts
- Traefik labels
- DNS
- backup coverage

## What Does Not Belong Here

Do not put large shared media under `/opt/data`.

Use NAS mounts or dedicated data disks for:

- movies
- TV shows
- photo originals
- large downloads

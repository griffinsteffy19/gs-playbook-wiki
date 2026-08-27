# Synology Storage Architecture

## Roles

The Synology NAS is more than a media server. It is a central infrastructure component.

Known roles include:

- media storage
- photo storage
- backup target
- Docker registry storage
- WikiMD host
- rsync server
- CIFS / SMB file server
- home directories

## Backup Target

Known rsync module:

```text
backup
```

Known target path:

```text
/volume1/homes/hometeam/backups/
```

Service backup snapshots land under:

```text
/volume1/homes/hometeam/backups/<node>/<service>/<timestamp>/
```

## Docker Registry

The NAS hosts the private registry backend.

Known access pattern:

```text
LAN:
nas.gs:5050

proxied:
registry.local.<domain>
```

## WikiMD

WikiMD is hosted on the NAS.

Known port:

```text
9454
```

## SMB / CIFS

Linux systems use NAS shares for bulk storage.

Known mounted data categories include:

- movies
- shows
- photos

Common Linux-side paths include:

```text
/data/movies
/data/shows
/data/photos
```

See [CIFS / SMB Mount Standards](cifs-smb-mount-standards) for `/etc/fstab`
patterns, credentials handling, and mount-failure diagnosis.

## Storage Layers

When diagnosing disk pressure, distinguish between:

1. guest filesystem usage
2. guest virtual-disk allocation
3. Proxmox thin-pool usage
4. NAS share usage

These are related but not equivalent. For example, `df -h` inside a VM may
show plenty of free filesystem space even while the Proxmox thin pool
remains heavily allocated. See [VM Disk, Discard, and TRIM](../core-infrastructure/vm-disk-and-trim)
for the full explanation and reclaim procedure.

### Useful Commands

Filesystem usage:

```bash
df -h
df -i
findmnt
mount
```

Largest top-level directories:

```bash
sudo du -xhd1 / | sort -h
sudo du -xhd1 /var | sort -h
```

Docker disk usage:

```bash
docker system df
docker system df -v
```

## Operational Notes

Synology CLI behavior varies by DSM version. Commands such as
`synoservicectl` / `synosystemctl` may not exist or behave consistently.
For service restarts, the DSM Control Panel UI can be more reliable.

Docker commands on the NAS may require `sudo` depending on user/group
permissions.

## Storage Placement Guidance

### Prefer local VM storage for

- application databases
- configuration
- latency-sensitive working data
- frequently accessed application metadata

### Prefer NAS storage for

- original media
- photos / videos
- shared media libraries
- backups
- archives
- large datasets that do not require local-block performance

### Avoid expensive backup treatment for regenerable data

Examples: thumbnails, encoded/transcoded media, caches, temporary downloads.

## Risks

Because the NAS holds both primary data and backups, consider whether
critical data also has an independent copy outside the NAS.

Questions to document:

- Is there an offsite copy?
- Is there a USB/offline copy?
- Are NAS snapshots enabled?
- Are backup credentials isolated?
- Can ransomware on a client delete backup copies?

## To Add

- [ ] NAS volume names and capacities
- [ ] SMB share inventory
- [ ] mount definitions
- [ ] backup-retention storage estimates
- [ ] SMART / array health monitoring procedure

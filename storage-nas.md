# Storage & NAS

## Synology NAS

The Synology NAS is a central storage system for the homelab.

Known uses include:

- movies
- TV shows
- photos
- home directories
- backup target
- Docker registry data
- WikiMD
- shared application storage

## Common Linux Mounts

Previously referenced mount paths include:

```text
/data/movies
/data/shows
/data/photos
```

Several Linux systems use CIFS / SMB mounts to the NAS.

## Storage Layers

When diagnosing disk pressure, distinguish between:

1. guest filesystem usage
2. guest virtual-disk allocation
3. Proxmox thin-pool usage
4. NAS share usage

These are related but not equivalent.

For example:

```bash
df -h
```

inside a VM may show lots of free filesystem space even while the Proxmox
thin pool remains heavily allocated.

## Useful Commands

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
```

For `/var`:

```bash
sudo du -xhd1 /var | sort -h
```

Docker disk usage:

```bash
docker system df
docker system df -v
```

## TRIM / Thin Provisioning

After deleting large amounts of local VM data:

```bash
sudo fstrim -av
```

Then verify reclaimed space on the Proxmox host with:

```bash
lvs
pvesm status
```

## Placement Guidance

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

Examples:

- thumbnails
- encoded/transcoded media
- caches
- temporary downloads

## To Add

- [ ] NAS volume names and capacities
- [ ] SMB share inventory
- [ ] mount definitions
- [ ] backup-retention storage estimates
- [ ] SMART / array health monitoring procedure

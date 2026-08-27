# Proxmox Storage Runbook

## Purpose

This page is a hands-on runbook for diagnosing Proxmox storage pressure and VM `io-error`
conditions.

## First Check: Host Storage

```bash
pvesm status
```

Then:

```bash
lvs
```

Pay close attention to:

```text
Data%
Meta%
```

for LVM-thin pools.

## Second Check: Guest Filesystem

Inside the VM:

```bash
df -h
df -i
```

These answer different questions:

```text
df -h -> free filesystem blocks
df -i -> free inodes
```

## Third Check: Large Directories

```bash
sudo du -xhd1 / | sort -h
```

Then drill down:

```bash
sudo du -xhd1 /var | sort -h
sudo du -xhd1 /opt | sort -h
sudo du -xhd1 /data | sort -h
```

For Docker:

```bash
docker system df
docker system df -v
```

## Thin-Pool Problem Pattern

See [VM Disk, Discard, and TRIM](vm-disk-and-trim) for the full
explanation of why deleting guest files alone is not enough to reclaim
thin-pool space.

## Reclaim Procedure

### 1. Delete known-safe data

Examples:

- generated thumbnails
- transcodes
- package caches
- unused Docker images
- temporary downloads

### 2. Confirm guest free space

```bash
df -h
```

### 3. Run TRIM

```bash
sudo fstrim -av
```

### 4. Re-check host thin pool

```bash
lvs
pvesm status
```

## Avoid Blind Cleanup

Do not immediately run destructive cleanup commands without understanding what they remove.

Examples requiring care:

```bash
docker system prune -a
rm -rf
```

Prefer targeted cleanup.

## Recommended Thresholds

| Thin Pool Data% | Action |
|---|---|
| `< 70%` | Normal |
| `70–85%` | Trend monitoring |
| `85–90%` | Investigate |
| `90–95%` | Immediate cleanup / expansion planning |
| `> 95%` | Critical |
| `~100%` | Expect VM I/O failure |

## VM Disk Settings

Check:

```bash
qm config <vmid>
```

Useful disk options:

```text
discard=on
ssd=1
iothread=1
```

depending on storage/backend.

## Post-Incident Tasks

After recovery:

- document what grew
- add monitoring
- decide whether the data belongs on NAS
- enable discard
- verify fstrim.timer
- review backup scope
- review VM disk sizing

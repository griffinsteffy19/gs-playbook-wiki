# Immich Operations and Storage

## Overview

Immich has been one of the largest storage consumers in the homelab.

It has run on the `record-book` Ubuntu VM.

## Known Issues

### v3.0.1 upgrade: "No vector extension found"

Hit when upgrading from a v2 stable release to v3.0.1.

**Cause:** Immich v3 migrated off the deprecated `pgvecto.rs` Postgres
extension to its successor, VectorChord (`vchord`). If the compose file
still references the old `tensorchord/pgvecto-rs` Postgres image instead
of `ghcr.io/immich-app/postgres`, the v3 server looks for `vchord` and
won't fall back to the old `vector` extension — hence the error listing
`vchord, vector` as "available" while still failing.

**Fix:**
1. **Back up the database first** — this touches the DB extension itself.
2. Update the compose file's Postgres image to `ghcr.io/immich-app/postgres`.
3. Do **not** set `DB_VECTOR_EXTENSION` manually unless intentionally
   pinning — if unset, Immich handles the VectorChord migration itself.

Reference: [Immich upgrading docs](https://docs.immich.app/install/upgrading/)

## Important Data Categories

Immich data should be separated into:

### Originals

Irreplaceable source photo / video files. These require strong backup coverage.

### Database / Application State

Contains user, album, indexing, and application metadata. Also requires backup.

### Derived Data

Examples: thumbnails, encoded video, generated previews. Large but
generally regenerable.

## Previous Storage Pressure

A previous incident included roughly:

```text
thumbs           ~26 GB
encoded-video   ~150 GB
```

The derived data was deleted to recover guest space. However, deleting
files did not immediately free equivalent Proxmox thin-pool allocation.
Running `sudo fstrim -av` reclaimed a large amount of thin-provisioned
storage — see [VM Disk, Discard, and TRIM](../core-infrastructure/vm-disk-and-trim).

## Preferred Architecture

```text
record-book local storage
    |
    +-- OS
    +-- Immich containers
    +-- database
    +-- app config
    +-- active metadata

Synology NAS
    |
    +-- original photos
    +-- original videos
    +-- backup copies
```

Derived assets may remain local if performance benefits justify it.

## Backup Policy

### Must back up

- originals
- database
- Compose/config files
- application settings
- secrets required for recovery

### Usually do not need full backup

- thumbnails
- encoded videos
- generated caches

## Recovery Strategy

If derived data is lost:

1. restore database / configuration
2. reconnect original library
3. start Immich
4. allow Immich to regenerate derived assets

## Monitoring

```bash
df -h
docker system df
du -sh <immich-data-paths>
lvs
pvesm status
```

Do not rely on only one layer of disk monitoring.

## To Do

- [ ] Document full compose file / storage layout
- [ ] Note whether Immich is included in [Service Backup](../storage-backup/service-backup)
      or needs its own backup strategy (likely needs DB dump, not just
      config-folder rsync, given the Postgres dependency)

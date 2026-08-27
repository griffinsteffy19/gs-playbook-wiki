# CIFS / SMB Mount Standards

## Overview

Linux VMs and Docker hosts use the Synology NAS for shared storage.

Common Linux-side mount points include:

```text
/data/movies
/data/shows
/data/photos
```

## Goals

Mounts should be:

- deterministic
- available before dependent applications start
- recoverable after temporary NAS outages
- documented
- protected from accidental writes to an empty local mount directory

## `/etc/fstab` Pattern

Use a dedicated credentials file rather than embedding credentials directly in `/etc/fstab`.

Conceptual example:

```fstab
//nas/share /data/share cifs credentials=/root/.smbcredentials,vers=3.0,_netdev,nofail 0 0
```

Additional options may be needed depending on ownership and application expectations.

## Credentials File

Example:

```text
/root/.smbcredentials
```

Permissions:

```bash
sudo chmod 600 /root/.smbcredentials
```

Do not document the actual username/password in WikiMD.

## `_netdev`

Use:

```text
_netdev
```

to identify the mount as network-dependent.

## `nofail`

`nofail` allows the machine to boot even if the NAS is unavailable.

This is useful for resilience, but creates an important hazard:

```text
application starts
mount missing
application writes into local empty directory
```

That can fill the VM disk unexpectedly.

## Recommended Service Guard

For critical services, consider requiring the mount before service start.

Systemd concept:

```text
RequiresMountsFor=/data/movies /data/shows
```

or check mount state in service startup logic.

## Validate Mount

```bash
findmnt /data/movies
mountpoint /data/movies
```

Return code check:

```bash
mountpoint -q /data/movies
```

## Diagnose Failure

```bash
dmesg | tail -50
journalctl -b | grep -i cifs
findmnt
mount
```

Known previous CIFS log:

```text
CIFS: Unable to determine destination address
```

This often points toward:

- DNS resolution
- server address issue
- temporary network failure

## Recovery

If the NAS becomes reachable again:

```bash
sudo mount -a
```

Then verify:

```bash
findmnt
df -h
```

## Docker Interaction

Never assume a bind mount is valid merely because the host path exists.

Before starting media services after a NAS outage:

```bash
mountpoint /data/movies
mountpoint /data/shows
```

## Documentation Template

| Mount | NAS Share | Used By | Required at Boot? | Credentials File |
|---|---|---|---|---|
| `/data/movies` | | Plex/Radarr | Yes | |
| `/data/shows` | | Plex/Sonarr | Yes | |
| `/data/photos` | | Immich | Yes | |

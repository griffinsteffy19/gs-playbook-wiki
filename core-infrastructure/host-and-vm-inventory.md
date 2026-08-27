# Host and VM Inventory

## Purpose

This page is a working inventory of known physical hosts, VMs, and workload names.

Only confirmed details are included. Unknown values are intentionally left blank rather
than guessed.

## Physical / Proxmox Hosts

| Name | Type | Known Role | Hardware / Notes |
|---|---|---|---|
| `gm` | Proxmox node | General virtualization | Dell OptiPlex 7070, Intel I219-LM NIC |
| `st-coordinator` | Proxmox node | VM host | Hosted VM 105 during storage incident |
| `o-coordinator` | Proxmox node | VM host | VM 104 storage work performed here |
| `fullback` | Proxmox node | **fill in** | Known cluster node name |
| `cold-plunge` | Proxmox node | **fill in** | Known cluster node name |
| `blue-tent` | Proxmox node | **fill in** | Known cluster node name |
| `punter` | Proxmox node | **fill in** | Known cluster node name |
| `kidney` | Proxmox node | **fill in** | Known cluster node name |

## VM / Host Names

### `record-book`

Known as an Ubuntu / Docker application host.

Known or previously associated workloads:

- Immich
- Vaultwarden
- Portainer CE

Storage model has included:

- OS disk
- separate large `/data` disk
- NAS-backed mounts

### `holder`

Known Linux / Docker host.

Known services:

- `backup`
- `gluetun`
- `qbittorrent`
- `prowlarr`
- `radarr`
- `sonarr`
- `tautulli`
- `kometa`
- `seerr`

Service configuration convention:

```text
/opt/data/<service>
```

## Known VM IDs

### VM 104

Associated with storage configuration work on `o-coordinator`.

Known disk examples included:

```text
scsi0 -> local-lvm
scsi1 -> Red4TB
```

At one point VM config showed:

```text
memory: 14336
cores: 1
cpu: host
name: record-book
```

Confirm whether VM 104 is still the canonical ID for `record-book` before treating this
as permanent inventory.

### VM 105

Previously associated with the `st-coordinator` thin-pool incident.

Known configuration at the time:

```text
cores: 8
memory: 20 GB
scsi0: 64 GB
scsi1: 512 GB
virtio-scsi-single
iothread=1
```

## Inventory Template

Use this format for each VM:

| Field | Value |
|---|---|
| VM ID | |
| Name | |
| Proxmox node | |
| Purpose | |
| vCPU | |
| Memory | |
| OS disk | |
| Data disk(s) | |
| Network / VLAN | |
| Static IP | |
| DNS name | |
| Backup policy | |
| Startup order | |
| Criticality | |
| Notes | |

## Recommended Next Step

Build one canonical matrix:

```text
physical node
  -> VM ID
    -> hostname
      -> IP
        -> workload
          -> backup policy
            -> storage backend
```

That becomes the authoritative infrastructure inventory.

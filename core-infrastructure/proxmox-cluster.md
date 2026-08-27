# Proxmox Cluster

## Overview

Primary cluster name: `Hometeam`

Full node inventory (names, roles, hardware) lives in
[Host and VM Inventory](host-and-vm-inventory) — this page covers
cluster-wide conventions only.

## Useful Commands

Cluster / storage:

```bash
pvesm status
lvs
```

VM configuration:

```bash
qm config <vmid>
qm status <vmid>
```

Boot logs:

```bash
journalctl -b -1
dmesg
```

## Virtual Disk Conventions

VirtIO SCSI is preferred for Linux VM disks where appropriate. Enable
discard where supported:

```bash
qm set <vmid> --scsi0 <storage>:<disk>,discard=on
```

For SSD-backed storage, `ssd=1` may also be appropriate.

See [VM Disk, Discard, and TRIM](vm-disk-and-trim) for the full
explanation of why discard/TRIM matters and the reclaim procedure — this
has directly mattered during previous thin-pool incidents (see
[Incident Notes](../operations/incident-notes)).

## Thin Pool Monitoring

| Data% | Guidance |
|---|---|
| `< 70%` | Healthy |
| `70–85%` | Watch trend |
| `85–90%` | Investigate growth |
| `90–95%` | Correct before planned growth |
| `> 95%` | Critical |
| `~100%` | VM I/O failures likely |

Full diagnostic/reclaim steps: [Proxmox Storage Runbook](proxmox-storage-runbook).

## Node: `gm`

Hardware and outage history documented on
[gm Proxmox Node](gm-proxmox-node) and [Incident Notes](../operations/incident-notes).

## To Add

- [ ] Node IP addresses
- [ ] VM ID → hostname → node mapping
- [ ] VM backup policy
- [ ] Storage backend by node
- [ ] HA / migration eligibility by VM

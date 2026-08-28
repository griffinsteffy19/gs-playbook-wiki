# Proxmox Cluster

## Overview

Primary cluster name: `Hometeam`

Full node inventory (names, roles, hardware) lives in

[Host and VM Inventory](host-and-vm-inventory) — this page covers

cluster-wide conventions only.

The cluster currently consists of three voting nodes:

* `gm`
* `st-coordinator`
* `o-coordinator`

With all three nodes participating, loss of a single node still leaves the cluster with quorum.

## High Availability

Selected VMs are managed by Proxmox HA and use shared Synology NFS storage so they can be restarted automatically on another cluster node if their preferred host fails.

Current protected guests:

| VM    | Name      | Preferred Host | Primary Failover | Secondary Failover |
| ----- | --------- | -------------- | ---------------- | ------------------ |
| `100` | `locker`  | `gm`           | `o-coordinator`  | `st-coordinator`   |
| `103` | `hass.gs` | `gm`           | `st-coordinator` | `o-coordinator`    |

Shared HA datastore:

```text
Storage ID:  synology-ha
Type:        NFS 4.1
Server:      10.10.0.20
Export:      /volume1/proxmox-ha
Purpose:     Shared VM disks for HA-managed guests
```

Both VMs have had their active disks migrated from `gm`'s local `local-lvm` storage to `synology-ha`.

The HA groups use `nofailback=1`, so after a failure the recovered guests remain on their failover nodes when `gm` returns. They are moved back manually after the node has been inspected.

A complete `gm` node-loss test has been performed successfully:

```text
gm DOWN

st-coordinator
├── VM105 holder
└── VM103 hass.gs

o-coordinator
├── VM104 record-book
└── VM100 locker
```

The surviving two nodes retained quorum and both protected VMs recovered automatically on their intended failover nodes.

Home Assistant has host-local USB passthrough devices on `gm`. During failover to another node, Home Assistant itself continues to run, but those USB devices are unavailable until the VM returns to `gm`.

See [Proxmox High Availability](proxmox-high-availability) for the complete architecture, HA group configuration, shared-storage setup, testing results, USB caveats, recovery workflow, and operational commands.

## Useful Commands

Cluster / storage:

```bash
pvesm status

lvs
```

HA status:

```bash
ha-manager status
```

HA configuration:

```bash
cat /etc/pve/ha/resources.cfg

cat /etc/pve/ha/groups.cfg
```

VM configuration:

```bash
qm config <vmid>

qm status <vmid>
```

Relocate an HA-managed VM:

```bash
ha-manager relocate vm:<vmid> <node>
```

HA logs:

```bash
journalctl -u pve-ha-crm

journalctl -u pve-ha-lrm
```

Follow HA activity live:

```bash
journalctl -f -u pve-ha-crm -u pve-ha-lrm
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

VMs intended for automatic HA recovery must not depend on node-local storage unless a supported replication strategy is configured.

Current HA-managed guests use the shared `synology-ha` NFS datastore.

See [VM Disk, Discard, and TRIM](vm-disk-and-trim) for the full

explanation of why discard/TRIM matters and the reclaim procedure — this

has directly mattered during previous thin-pool incidents (see

[Incident Notes](../operations/incident-notes)).

## Thin Pool Monitoring

| Data%    | Guidance                      |
| -------- | ----------------------------- |
| `< 70%`  | Healthy                       |
| `70–85%` | Watch trend                   |
| `85–90%` | Investigate growth            |
| `90–95%` | Correct before planned growth |
| `> 95%`  | Critical                      |
| `~100%`  | VM I/O failures likely        |

Full diagnostic/reclaim steps: [Proxmox Storage Runbook](proxmox-storage-runbook).

## Node: `gm`

Hardware and outage history documented on

[gm Proxmox Node](gm-proxmox-node) and [Incident Notes](../operations/incident-notes).

`gm` is currently the preferred host for:

* VM100 `locker`
* VM103 `hass.gs`

Both guests are HA-managed and can recover automatically to the remaining cluster nodes if `gm` becomes unavailable.

## Related Pages

* [Host and VM Inventory](host-and-vm-inventory)
* [Proxmox High Availability](proxmox-high-availability)
* [Proxmox Storage Runbook](proxmox-storage-runbook)
* [VM Disk, Discard, and TRIM](vm-disk-and-trim)
* [gm Proxmox Node](gm-proxmox-node)
* [Incident Notes](../operations/incident-notes)

## To Add

* [ ] Node IP addresses
* [ ] Complete VM ID → hostname → preferred node mapping
* [ ] VM backup policy
* [ ] Storage backend by node
* [ ] Document Synology NFS permissions after restricting `synology-ha` from `10.10.0.0/24` to the individual Proxmox nodes

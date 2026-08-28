# Proxmox High Availability

## Overview

The Hometeam Proxmox cluster is configured to support automatic recovery of selected VMs if their normal host, `gm`, becomes unavailable.

The design intentionally spreads failover load across the two surviving nodes rather than attempting to place both protected VMs on the same host.

### Cluster Nodes

| Node | CPU | RAM | HA Role |
|---|---|---:|---|
| `gm` | Intel Core i5-9500T, 6 cores | 32 GB | Preferred host for VM100 and VM103 |
| `st-coordinator` | AMD Ryzen 7 6800U, 8C/16T | 20 GB | Preferred failover host for Home Assistant |
| `o-coordinator` | Intel Core i3-2100, 2 cores | 16 GB | Preferred failover host for `locker`; emergency HA target |

The cluster has three votes. If one node is lost, the remaining two nodes retain quorum and can continue HA operations.

---

## Protected VMs

### VM100 — `locker`

Normal placement:

```text
gm
└── VM100 locker
```

Preferred failover order:

```text
gm
  ↓
o-coordinator
  ↓
st-coordinator
```

Resources:

```text
2 vCPU
6147 MB RAM
Ubuntu 22.04
```

Storage:

```text
efidisk0 → synology-ha
ide2     → synology-ha (cloud-init)
scsi0    → synology-ha (~26 GB)
```

VM100 has no USB or PCI passthrough and is therefore the simpler of the two HA guests.

---

### VM103 — `hass.gs`

Normal placement:

```text
gm
└── VM103 Home Assistant OS
```

Preferred failover order:

```text
gm
  ↓
st-coordinator
  ↓
o-coordinator
```

Resources:

```text
2 vCPU
8192 MB RAM
CPU type: host
Home Assistant OS
```

Storage:

```text
efidisk0 → synology-ha
scsi0    → synology-ha (32 GB)
```

Host-local USB passthrough:

```text
usb0: host=10c4:ea60
usb1: host=2357:0604
```

`2357:0604` is the TP-Link Bluetooth adapter.

`10c4:ea60` is a Silicon Labs CP210x USB-to-serial device and is used by a host-local Home Assistant peripheral.

These USB devices remain physically attached to `gm`.

If Home Assistant fails over to another node:

```text
Home Assistant OS             available
Network integrations          available
Automations                   available
Synology/network resources    available
Host-local USB devices        unavailable
```

This degraded failover mode has been tested successfully on `st-coordinator`.

A manual Proxmox migration with these USB mappings requires:

```bash
qm migrate 103 st-coordinator --force
```

HA-managed relocation was also successfully tested with the USB mappings still present.

---

# Shared HA Storage

## Synology NFS Datastore

HA VM disks are stored on a dedicated Synology NFS shared folder.

```text
Synology host:       longsnapper.gs
Synology IP:         10.10.0.20
NFS export:          /volume1/proxmox-ha
Proxmox storage ID:  synology-ha
Content:             VM disk images
NFS version:         4.1
```

Proxmox configuration:

```bash
pvesm add nfs synology-ha \
  --server 10.10.0.20 \
  --export /volume1/proxmox-ha \
  --content images \
  --options vers=4.1
```

The storage definition is cluster-wide through `/etc/pve/storage.cfg`.

Verify from every node:

```bash
pvesm status | grep synology-ha
```

Expected:

```text
synology-ha    nfs    active
```

The same NFS filesystem has been verified from:

```text
gm
st-coordinator
o-coordinator
```

### NFS Permission

During initial setup, the Synology export permits:

```text
10.10.0.0/24
```

Recommended future hardening:

```text
Restrict the NFS permission to only:
- gm
- st-coordinator
- o-coordinator
```

rather than the full LAN subnet.

---

## NFS Performance

Measured from `gm`:

### Sequential Write

```bash
dd if=/dev/zero \
   of=/mnt/pve/synology-ha/io-test \
   bs=1M count=1024 \
   conv=fdatasync
```

Result:

```text
1 GiB written in 9.72 seconds
~111 MB/s
```

This is approximately the practical limit of a 1 GbE connection.

### Proxmox Filesystem Test

```bash
pveperf /mnt/pve/synology-ha
```

Measured:

```text
FSYNCS/SECOND: 873
```

This was considered sufficient for the current Home Assistant workload.

---

# Local VM Storage vs Shared Storage

Before HA:

```text
gm local-lvm
├── VM100
└── VM103
```

A VM stored only on `local-lvm` cannot automatically start on another cluster node because the destination cannot access its disks.

After migration:

```text
                   Synology
                synology-ha
                     │
          ┌──────────┼──────────┐
          │          │          │
         gm   st-coordinator  o-coordinator
```

VM100 and VM103 now reference only shared NFS storage for their active VM disks.

Old `local-lvm` copies were retained temporarily as cold rollback copies during extended testing.

Important:

```text
An unused local disk still referenced as `unusedX:` in the VM
configuration is considered part of that VM during migration.
```

For extended HA testing, old rollback disks may remain physically present on `gm`, but the `unusedX:` references should be removed from the active VM configuration so Proxmox does not attempt to migrate them.

Do not destroy those volumes until the shared-storage configuration has been proven stable for an acceptable period.

---

# HA Capacity Planning

The failover policy intentionally splits VM100 and VM103 between the surviving hosts.

After right-sizing the existing VMs:

## `st-coordinator`

VM105 `holder`:

```text
Maximum RAM: 12 GB
Balloon minimum: 8 GB
```

Typical guest usage was approximately:

```text
3.3 GB
```

After reboot/right-sizing, the host had approximately:

```text
13 GB available
```

This makes it the preferred recovery node for the 8 GB Home Assistant VM.

## `o-coordinator`

VM104 `record-book`:

```text
Maximum RAM: 8 GB
Balloon minimum: 6 GB
```

Typical Immich workload was approximately:

```text
2.2 GB guest RAM used
```

After reboot/right-sizing, the host had approximately:

```text
11 GB available
```

Although its i3-2100 CPU is much slower, it has sufficient emergency capacity for VM100.

---

# HA Groups

Two different HA groups are used so a failure of `gm` naturally distributes the workloads.

## Home Assistant Group

```bash
ha-manager groupadd hass-failover \
  --nodes 'gm:3,st-coordinator:2,o-coordinator:1' \
  --restricted 1 \
  --nofailback 1
```

Priority:

```text
1. gm
2. st-coordinator
3. o-coordinator
```

## Locker Group

```bash
ha-manager groupadd locker-failover \
  --nodes 'gm:3,o-coordinator:2,st-coordinator:1' \
  --restricted 1 \
  --nofailback 1
```

Priority:

```text
1. gm
2. o-coordinator
3. st-coordinator
```

## HA Resources

```bash
ha-manager add vm:103 \
  --group hass-failover \
  --state started

ha-manager add vm:100 \
  --group locker-failover \
  --state started
```

Verify:

```bash
ha-manager status
```

Typical healthy state while both VMs are on `gm`:

```text
quorum OK
master <node> (active)
lrm gm (...)
lrm o-coordinator (...)
lrm st-coordinator (...)

service vm:100 (gm, started)
service vm:103 (gm, started)
```

---

# `nofailback`

Both groups use:

```text
nofailback=1
```

If `gm` fails and the VMs recover elsewhere:

```text
VM103 → st-coordinator
VM100 → o-coordinator
```

when `gm` returns, the VMs remain on their recovery nodes.

They are not automatically moved back.

This is intentional.

It provides time to:

1. Verify `gm` is healthy.
2. Inspect the cause of the outage.
3. Confirm storage and networking are stable.
4. Move workloads back deliberately.

Return them with:

```bash
ha-manager relocate vm:103 gm
ha-manager relocate vm:100 gm
```

---

# HA Services

Each HA-capable node needs the Proxmox HA daemons running.

Important services:

```text
pve-ha-crm
pve-ha-lrm
watchdog-mux
```

`o-coordinator` originally had the HA services disabled.

They were enabled with:

```bash
systemctl enable pve-ha-lrm pve-ha-crm
systemctl start pve-ha-lrm
systemctl start pve-ha-crm
```

Verify:

```bash
systemctl status pve-ha-lrm
systemctl status pve-ha-crm
systemctl status watchdog-mux
```

If a node is missing from HA status with an error such as:

```text
unable to read file '/etc/pve/nodes/<node>/lrm_status'
```

check `pve-ha-lrm` on that node first.

---

# Controlled Relocation

Once a VM is managed by HA, use `ha-manager` rather than normal `qm migrate` for routine HA placement.

Home Assistant:

```bash
ha-manager relocate vm:103 st-coordinator
```

Return it:

```bash
ha-manager relocate vm:103 gm
```

Locker:

```bash
ha-manager relocate vm:100 o-coordinator
```

Return it:

```bash
ha-manager relocate vm:100 gm
```

Watch state transitions:

```bash
watch -n 2 'date; echo; ha-manager status'
```

---

# Validation Completed

## Shared Storage

Verified:

```text
[✓] synology-ha active on gm
[✓] synology-ha active on st-coordinator
[✓] synology-ha active on o-coordinator
[✓] file created on one node visible on the others
```

## VM100

Verified:

```text
[✓] Active VM disks moved from local-lvm to synology-ha
[✓] VM boots successfully from shared storage
```

## VM103

Verified:

```text
[✓] Active VM disks moved from local-lvm to synology-ha
[✓] VM boots successfully from shared storage
[✓] Manual migration gm → st-coordinator works with --force
[✓] HAOS operates on st-coordinator without the gm USB devices
[✓] Migration back to gm succeeds
[✓] USB functionality returns on gm
[✓] HA-manager relocation to st-coordinator succeeds
```

## HA Cluster

Verified:

```text
[✓] Cluster quorum healthy
[✓] HA CRM active
[✓] LRM active/idle on all three nodes
[✓] VM100 registered as HA resource
[✓] VM103 registered as HA resource
[✓] Full gm node-loss recovery test completed successfully
[✓] VM103 recovered automatically to st-coordinator
[✓] VM100 recovered automatically to o-coordinator
```

---

# Full Node-Failure Test

A complete `gm` node-failure test was performed successfully.

## Test Procedure

Before the test, both protected VMs were running normally on `gm`:

```text
VM100 locker  → gm
VM103 hass.gs → gm
```

HA status was monitored from a surviving node with:

```bash
watch -n 2 'date; echo; ha-manager status'
```

and HA logs could also be followed with:

```bash
journalctl -f -u pve-ha-crm -u pve-ha-lrm
```

`gm` was then shut down without manually relocating VM100 or VM103 first.

## Result

The remaining two nodes retained quorum and Proxmox HA recovered both guests automatically.

Observed recovery placement matched the intended design:

```text
gm DOWN

st-coordinator
├── VM105 holder
└── VM103 hass.gs

o-coordinator
├── VM104 record-book
└── VM100 locker
```

Validation result:

```text
[✓] gm loss detected by HA
[✓] Cluster retained quorum with 2 of 3 nodes
[✓] VM103 automatically recovered on st-coordinator
[✓] VM100 automatically recovered on o-coordinator
[✓] Home Assistant booted successfully without gm-local USB devices
[✓] Shared synology-ha storage remained available
[✓] HA recovery priorities behaved as designed
```

After `gm` returned, `nofailback=1` prevented an automatic move back to `gm`, allowing the node to be inspected before workloads were manually relocated.

This validates the HA design end-to-end for a complete loss of `gm`.

---

# Failure Dependencies

This HA design protects against a Proxmox compute-node failure.

It does **not** protect against every infrastructure failure.

## Synology Failure

Both protected VMs depend on:

```text
10.10.0.20:/volume1/proxmox-ha
```

If the Synology or its NFS service is unavailable, the VM disks are unavailable to every node.

Therefore:

```text
Proxmox node failure    → protected
gm hardware failure    → protected
gm OS failure          → protected
Synology failure       → NOT protected by this HA design
LAN failure            → may prevent recovery
```

Future consideration:

```text
ZFS replication
```

could keep local VM copies on multiple Proxmox nodes and remove the Synology from the VM-runtime dependency chain, but the current nodes use LVM-thin and would require a storage redesign.

---

# Operational Commands

## Overall HA Status

```bash
ha-manager status
```

## HA Configuration

```bash
cat /etc/pve/ha/resources.cfg
cat /etc/pve/ha/groups.cfg
```

## Shared Storage

```bash
pvesm status | grep synology-ha
```

## VM Placement

```bash
ha-manager status | grep 'vm:'
```

## Relocate Home Assistant

```bash
ha-manager relocate vm:103 <node>
```

## Relocate Locker

```bash
ha-manager relocate vm:100 <node>
```

## HA Logs

```bash
journalctl -u pve-ha-crm
journalctl -u pve-ha-lrm
```

Follow live:

```bash
journalctl -f -u pve-ha-crm -u pve-ha-lrm
```

---

# Current Architecture

```text
                             ┌───────────────────────────┐
                             │ Synology longsnapper.gs   │
                             │ 10.10.0.20                │
                             │                           │
                             │ /volume1/proxmox-ha       │
                             │      synology-ha          │
                             │                           │
                             │ VM100 disks               │
                             │ VM103 disks               │
                             └─────────────┬─────────────┘
                                           │ NFS 4.1
                   ┌───────────────────────┼────────────────────────┐
                   │                       │                        │
                   ▼                       ▼                        ▼
          ┌─────────────────┐    ┌──────────────────┐     ┌──────────────────┐
          │ gm              │    │ st-coordinator   │     │ o-coordinator    │
          │ i5-9500T        │    │ Ryzen 7 6800U    │     │ i3-2100          │
          │ 32 GB RAM       │    │ 20 GB RAM        │     │ 16 GB RAM        │
          │                 │    │                  │     │                  │
Normal →  │ VM100 locker    │    │ VM105 holder     │     │ VM104 record-book│
Normal →  │ VM103 hass.gs   │    │                  │     │                  │
          └─────────────────┘    └──────────────────┘     └──────────────────┘
                   │                       ▲                        ▲
                   │ gm fails              │                        │
                   ├───────────────────────┘                        │
                   │        VM103 → st-coordinator                  │
                   │                                                │
                   └────────────────────────────────────────────────┘
                            VM100 → o-coordinator
```

---
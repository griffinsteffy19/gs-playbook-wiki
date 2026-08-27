# VM Disk, Discard, and TRIM

## Why This Matters

In a virtualized environment, "free space inside the VM" is not always the same as
"free space on the Proxmox host."

This distinction has directly mattered in the homelab.

## Layers

```text
Application
   |
Filesystem inside VM
   |
Virtual disk
   |
Proxmox storage
   |
Physical disk / thin pool
```

Deleting a file only tells the filesystem that blocks are unused.

The storage layer may continue to consider those virtual blocks allocated.

## Discard

Proxmox disk option:

```text
discard=on
```

allows the guest to issue discard/TRIM requests through the virtual disk.

Example:

```bash
qm set <vmid> --scsi0 <storage>:<disk>,discard=on
```

## TRIM in Linux

Manual:

```bash
sudo fstrim -av
```

Check timer:

```bash
systemctl status fstrim.timer
```

Enable if appropriate:

```bash
sudo systemctl enable --now fstrim.timer
```

## What TRIM Does

TRIM does not delete live files.

It tells lower storage layers:

```text
"these blocks are no longer in use by the filesystem"
```

This allows thin-provisioned storage to reclaim allocation.

## What TRIM Does Not Do

TRIM does not:

- compress data
- remove live files
- reduce logical filesystem size
- automatically shrink a partition
- replace backups

## Good Candidates

VMs with:

- LVM-thin-backed disks
- SSD-backed storage
- large generated datasets
- frequent deletions
- Docker workloads
- photo/video processing

## Verification

Inside guest before:

```bash
df -h
```

Run:

```bash
sudo fstrim -av
```

Then on Proxmox:

```bash
lvs
pvesm status
```

If thin-pool Data% falls, reclaimed blocks successfully propagated.

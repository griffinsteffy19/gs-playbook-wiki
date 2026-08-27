# Incident Notes

This page is for concise operational lessons from incidents, not full raw logs.

## Proxmox Thin-Pool Exhaustion

### Symptoms

A VM entered:

```text
io-error
```

while Proxmox LVM-thin usage was near capacity.

### Important Detail

Deleting files inside the guest does not necessarily return space to the host thin pool
until discard / TRIM is issued.

### Recovery Pattern

Inside the guest:

```bash
sudo fstrim -av
```

On the Proxmox host:

```bash
lvs
pvesm status
```

### Lesson

Monitor guest filesystem usage and host thin-pool usage independently.

---

## Immich Generated Data Growth

Immich produced large amounts of derived data.

Previously observed examples included large:

- thumbnail directories
- encoded-video directories

After deleting regenerable assets, TRIM was required to reclaim thin-provisioned blocks.

### Lesson

Separate:

- originals / irreplaceable data
- application state
- regenerable thumbnails / transcoded video / caches

and apply different backup / storage policies to each.

---

## `gm` Unexpected Shutdown

Host:

```text
gm
```

Hardware previously identified as:

```text
Dell OptiPlex 7070
Intel I219-LM
e1000e driver
```

Observed behavior:

- host became unreachable
- VMs went down
- physical power-on was required

Network troubleshooting included disabling:

- EEE
- Wake-on-LAN
- TSO
- GSO
- GRO

UniFi disconnect timestamps and previous-boot journal logs were reviewed.

No definitive root cause was established from the available evidence.

### Lesson

For future unexplained Proxmox outages, preserve and compare:

```text
journalctl -b -1
kernel logs
UniFi disconnect timestamp
BIOS event logs
power / UPS events
```

before reboot cycles overwrite useful context.

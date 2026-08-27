# Proxmox Node: gm

## Overview

`gm` is a Proxmox node in the `Hometeam` cluster.

Known hardware:

```text
Dell OptiPlex 7070
Intel I219-LM onboard NIC
PCI ID: 8086:15bb
driver: e1000e
```

Known BIOS version previously observed:

```text
1.35.0
2025-09-04
```

Known Proxmox/kernel context from troubleshooting:

```text
Proxmox VE 8.4.14
kernel 6.8.12-15-pve
```

## Network

Known bridge:

```text
vmbr0
```

Known physical interface:

```text
eno1
```

Known host address at the time of troubleshooting:

```text
10.10.0.100/24
gateway 10.10.0.1
```

The physical NIC is attached to `vmbr0`.

## Unexpected Shutdown Incident

A previous outage resulted in:

- host unreachable
- all VMs on `gm` unavailable
- physical power-button press required to recover
- UniFi showing the host disconnected

One correlated UniFi disconnect was observed around:

```text
2026-08-20T00:22:40Z
```

Earlier system logs also showed `eno1` link down/up events.

## NIC Troubleshooting

To reduce the chance of an e1000e / power-management-related problem, the following features
were tested disabled:

```text
EEE
Wake-on-LAN
TSO
GSO
GRO
```

Useful checks:

```bash
ethtool eno1
ethtool -k eno1
journalctl -k -b
journalctl -k -b -1
```

## Network Link Investigation

Search recent boots:

```bash
journalctl -b | grep -i -E 'eno1|e1000e|link'
journalctl -b -1 | grep -i -E 'eno1|e1000e|link'
```

Kernel-only view:

```bash
journalctl -k -b -1
```

## Power Investigation

If the node unexpectedly powers off again:

1. do not immediately assume network failure
2. correlate UniFi disconnect time with journal
3. inspect BIOS event logs if available
4. inspect AC power / UPS history
5. verify whether chassis power LED indicates sleep/off/fault
6. confirm whether the node can be woken remotely
7. preserve `journalctl -b -1` before additional reboots

## Expansion Constraints

The current small-form-factor / micro-class hardware does not provide a practical PCIe NIC
upgrade path.

If the onboard NIC proves unreliable, replacement options may involve:

- USB Ethernet for testing only
- replacing the system
- moving critical workloads to another Proxmox node

## Related Pages

- [Proxmox Cluster](proxmox-cluster)
- [Incident Notes](../operations/incident-notes)
- [Wake-on-LAN and Power Management](wake-on-lan-and-power)

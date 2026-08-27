# Hardware Inventory

## Purpose

Known physical hardware relevant to the homelab.

## Network

### Ubiquiti UDM Pro

Role:

- router
- firewall
- UniFi controller
- VLAN gateway
- WAN edge

## Storage

### Synology NAS

Role:

- SMB/CIFS
- media
- photos
- backups
- registry storage
- WikiMD

Exact model / drive layout: **fill in**

## Proxmox / Compute

### Dell OptiPlex 7070

Known hostname:

```text
gm
```

Known NIC:

```text
Intel I219-LM
PCI ID: 8086:15bb
driver: e1000e
```

Known BIOS previously observed:

```text
1.35.0
2025-09-04
```

This system has no practical PCIe expansion path in the current form factor.

### Beelink SER3

Known hardware:

```text
AMD Ryzen 3 3200U
16 GB DDR4
500 GB SSD
```

Used / available as lab compute.

### Beelink EQR7

Known hardware:

```text
AMD Ryzen 7 7735U
24 GB LPDDR5
500 GB SSD
```

Used for heavier lab workloads / services.

## Client / Admin Systems

### MacBook Pro

Known configuration:

```text
16-inch
M1 Max
32 GB RAM
2 TB SSD
```

Used for development / administration and photo/video editing.

## Hardware Inventory Template

| Host | Model | CPU | RAM | Storage | NIC | Role | OS | Notes |
|---|---|---|---|---|---|---|---|---|
| `gm` | Dell OptiPlex 7070 | | | | Intel I219-LM | Proxmox | Proxmox VE | |
| `holder` | | | | | | Docker/media | Ubuntu/Linux | |
| `record-book` | VM | | | | virtual | Docker/apps | Ubuntu | |

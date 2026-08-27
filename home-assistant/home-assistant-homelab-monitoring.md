# Home Assistant Homelab Monitoring

## Purpose

Home Assistant is used not only for home automation but also as an infrastructure overview.

## Known Integrations

Infrastructure-related integrations include:

- Proxmox VE
- Plex
- Tautulli
- UniFi Network
- Synology DSM
- Ambient Weather

Glances has also been used on Ubuntu hosts.

## Desired Dashboard Direction

The preferred direction is a single high-value operational page rather than a large,
room-style dashboard.

A useful infrastructure dashboard should answer:

```text
Is the network up?
Are Proxmox nodes healthy?
Are critical VMs running?
Is storage filling?
Did backups succeed?
Is Plex healthy?
Is the NAS healthy?
```

## Recommended Sections

### Critical Alerts

Show only actionable problems:

- VM down
- node unavailable
- thin pool high
- NAS degraded
- backup failed
- internet unavailable

### Proxmox

Per-node:

- online state
- CPU
- memory
- storage pressure

Per-critical VM:

- running state
- CPU
- memory

### Synology

Show:

- volume health
- utilization
- drive warnings

### UniFi

Show:

- WAN status
- gateway health
- key switch status
- disconnected infrastructure clients

### Plex / Tautulli

Show:

- Plex availability
- active streams
- transcodes
- bandwidth

### Linux Hosts

Through Glances:

- CPU
- RAM
- root filesystem
- load
- temperature if available

## Mobile Design Principle

Keep top-level cards small and actionable.

Use drill-downs / popups only for details.

The first screen should fit the most important state without scrolling excessively.

## Suggested Alert Thresholds

| Metric | Warning | Critical |
|---|---:|---:|
| Filesystem usage | 80% | 90% |
| Proxmox thin pool | 85% | 95% |
| RAM | 85% sustained | 95% |
| CPU | 85% sustained | 95% |
| VM state | n/a | stopped unexpectedly |

## Related Pages

- [Monitoring Overview](../operations/monitoring-overview)
- [Proxmox Storage Runbook](../core-infrastructure/proxmox-storage-runbook)

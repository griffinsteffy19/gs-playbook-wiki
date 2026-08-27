# Monitoring Overview

## Monitoring Sources

### Proxmox

Provides:

- host status
- VM status
- CPU / memory
- storage usage
- thin-pool utilization

### UniFi

Useful for:

- host disconnect timestamps
- switch-port state
- client connectivity
- traffic history

### Synology DSM

Provides NAS health and storage monitoring.

### Glances

Used on Ubuntu hosts for system metrics.

Glances has been run as a systemd service.

### Home Assistant

Acts as a convenient aggregation layer for several infrastructure integrations.

## Recommended High-Value Alerts

- Proxmox thin pool > 85%
- Proxmox thin pool > 95%
- local filesystem > 85%
- critical VM stopped
- failed VM backup
- NAS degraded
- Docker host unavailable
- Traefik unavailable
- Pi-hole unavailable
- Home Assistant unavailable

## Thin-Pool Alerting

Guest filesystem space and Proxmox thin-pool usage must be monitored separately.

A VM may show healthy free space while the host thin pool remains dangerously full.

## Dashboard Philosophy

Prefer one concise infrastructure overview showing:

- critical alerts
- storage pressure
- VM / host health
- backup status
- network state
- important service availability

Use service-specific dashboards only for deeper troubleshooting.

## To Add

- [ ] actual alert thresholds
- [ ] notification destinations
- [ ] host-by-host monitoring coverage
- [ ] NAS disk / SMART alerts
- [ ] backup-success sensors

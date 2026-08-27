# Glances Monitoring

## Overview

Glances is used on Ubuntu systems to expose host-level metrics.

It has been run as a systemd service.

## Useful Metrics

Monitor:

- CPU
- load average
- RAM
- swap
- root filesystem
- data filesystems
- process count
- temperatures
- network I/O
- disk I/O

## Systemd

Check service:

```bash
systemctl status glances
```

Logs:

```bash
journalctl -u glances
```

Restart:

```bash
sudo systemctl restart glances
```

Enable at boot:

```bash
sudo systemctl enable glances
```

## Home Assistant

Glances can feed Home Assistant host metrics.

This provides a useful complement to:

- Proxmox integration
- Synology integration
- UniFi integration

Proxmox shows the VM from outside.

Glances shows the guest OS from inside.

## Recommended Alerts

### Filesystem

```text
warning: >80%
critical: >90%
```

### Memory

Use sustained rather than instantaneous high RAM as the alert condition.

### CPU

Likewise, alert only if high utilization persists.

### Load

Interpret relative to core count.

## Host Coverage

Recommended for:

- Docker hosts
- important Ubuntu VMs
- physical Linux hosts not already deeply monitored elsewhere

## Troubleshooting

If HA stops receiving metrics:

1. check Glances service
2. check listening port
3. test connectivity from HA network
4. inspect firewall
5. check HA integration logs

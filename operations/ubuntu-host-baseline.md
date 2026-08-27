# Ubuntu Host Baseline

## Scope

Common baseline for Ubuntu 24.04 homelab systems.

## Core Packages

Useful operational packages include:

```text
curl
git
htop
iotop
lm-sensors
ethtool
rsync
cifs-utils
smartmontools
iperf3
fio
stress-ng
procps
```

Install only what is appropriate for the host.

## Updates

```bash
sudo apt update
sudo apt upgrade
```

For critical infrastructure, review major package changes before applying blindly.

## Time

Check:

```bash
timedatectl
```

Reliable time is important for:

- TLS
- logs
- authentication
- distributed debugging

## Disk

```bash
df -h
df -i
lsblk
findmnt
```

## Networking

```bash
ip addr
ip route
ss -tulpn
ethtool <interface>
```

## Logs

Current boot:

```bash
journalctl -b
```

Previous boot:

```bash
journalctl -b -1
```

Kernel:

```bash
journalctl -k
```

## fstrim

For virtual disks with discard support:

```bash
systemctl status fstrim.timer
sudo systemctl enable --now fstrim.timer
```

## Docker Log Rotation

Use bounded logs.

Example:

```yaml
logging:
  driver: json-file
  options:
    max-size: "10m"
    max-file: "3"
```

## Monitoring

Glances can be enabled as a systemd service for Home Assistant integration.

## Mount Safety

For NAS-backed services, verify network mounts before starting applications that expect them.

## Documentation

Each host page should include:

- hostname
- IP
- OS version
- role
- hardware
- Docker stacks
- storage
- NAS mounts
- backup coverage
- monitoring

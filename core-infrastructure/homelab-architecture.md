# Homelab Architecture

## Purpose

This page describes the homelab at an architectural level: how compute, storage,
networking, DNS, reverse proxying, authentication, backups, and applications fit together.

It is intentionally broader than the service-specific pages.

## High-Level Topology

```text
Internet
   |
   v
Ubiquiti UDM Pro
   |
   +-- Trusted LAN
   |
   +-- Isolated / TV VLAN
   |
   +-- Other VLANs / segments
   |
   v
Pi-hole DNS
   |
   +-- local host records
   +-- wildcard local DNS
   |
   v
Traefik v3
   |
   +-- public HTTPS services
   +-- internal HTTPS services
   +-- Authelia-protected services
   |
   +-----------------------------+
   |                             |
   v                             v
Docker hosts                 Proxmox cluster
(holder, record-book,        "Hometeam"
 other service nodes)            |
   |                             +-- Home Assistant OS
   |                             +-- Docker VMs
   |                             +-- application VMs
   |
   +-----------------------------+
                 |
                 v
             Synology NAS
                 |
                 +-- media
                 +-- photos
                 +-- backups
                 +-- registry data
                 +-- shared storage
```

## Major Components

### Network Edge

Primary routing and firewall platform:

```text
Ubiquiti UDM Pro
```

Responsibilities:

- Internet gateway
- VLAN routing
- firewall policy
- client visibility
- switch management
- network segmentation

### DNS

Primary internal DNS:

```text
Pi-hole
```

Pi-hole is used for:

- LAN hostname resolution
- local service records
- wildcard records
- ad / tracker filtering
- internal routing to Traefik-backed services

### Reverse Proxy

Primary reverse proxy:

```text
Traefik v3
```

Responsibilities:

- TLS termination
- host-based routing
- public HTTPS ingress
- internal HTTPS ingress
- middleware
- Authelia integration

### Authentication

Selected services use:

```text
Authelia
```

for SSO / MFA.

Not every application is necessarily behind Authelia. Some services have their own
authentication and should remain that way if proxy-layer authentication adds little value.

### Compute

Primary virtualization environment:

```text
Proxmox cluster: Hometeam
```

Known node names include:

- `gm`
- `st-coordinator`
- `o-coordinator`
- `fullback`
- `cold-plunge`
- `blue-tent`
- `punter`
- `kidney`

Separate Linux VMs and Docker hosts run application workloads.

### Storage

Primary centralized storage:

```text
Synology NAS
```

Used for:

- movies
- TV shows
- photos
- shared data
- backups
- Docker registry data
- WikiMD
- home directories

### Home Automation

Home Assistant OS runs as a VM on Proxmox.

It is also used as a monitoring surface for:

- Proxmox
- Plex
- Tautulli
- UniFi
- Synology
- Ambient Weather
- selected Linux-host metrics

## Design Principles

### Keep application config local and predictable

Preferred convention:

```text
/opt/data/<service>
```

### Centralize large shared data

Large media and shared datasets generally belong on the NAS.

### Separate regenerable from irreplaceable data

Examples of regenerable data:

- thumbnails
- encoded video
- transcodes
- caches
- Docker images

Examples of irreplaceable / high-value data:

- databases
- configuration
- Home Assistant state
- Vaultwarden
- original photos
- important documents
- custom scripts

### Use DNS names instead of memorized IP addresses

Services should be referenced through local DNS names wherever practical.

### Keep the wiki secret-free

Document locations and procedures, not credentials.

## Related Pages

- [Networking & DNS](../network-dns/networking-dns)
- [Proxmox Cluster](proxmox-cluster)
- [Synology Storage Architecture](../storage-backup/synology-storage-architecture)
- [Backup Strategy](../storage-backup/backup-strategy)
- [Reverse Proxy and Access Paths](../network-dns/reverse-proxy-access-paths)
- [Security Boundaries](../network-dns/security-boundaries)

# Home Assistant Backup and Recovery

## Overview

Home Assistant OS runs as a Proxmox VM.

It is one of the highest-priority systems in the homelab.

## Backup Layers

Home Assistant already has regular backup coverage.

Use two complementary layers:

### Home Assistant native backup

Good for:

- configuration
- integrations
- add-ons
- restore within HA

### Proxmox VM backup

Good for:

- full VM recovery
- catastrophic VM corruption
- moving/recovering the whole appliance

## Recommended Recovery Order

If Home Assistant VM is lost:

1. restore VM from Proxmox backup
2. boot HA
3. verify network
4. verify DNS
5. verify storage
6. verify integrations
7. if needed, restore latest HA-native backup
8. verify automations
9. verify mobile app / external URL

## Critical Integrations to Verify

Known important integrations include:

- Proxmox VE
- UniFi
- Synology DSM
- Plex
- Tautulli
- Ambient Weather
- thermostat / Ecobee
- Samsung Frame TV

## Cross-VLAN Dependencies

After restore, verify firewall rules still allow required device access.

## Secrets

Ensure `secrets.yaml` and integration tokens are preserved by backup.

Do not copy those values into WikiMD.

## Test Restore

Periodically document:

| Date | Backup Type | Restored To | Result |
|---|---|---|---|
| | HA native | | |
| | Proxmox VM | | |

## Failure Prevention

Monitor:

- VM running state
- backup success
- HA availability
- host storage usage
- Proxmox thin pool

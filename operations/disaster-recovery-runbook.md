# Disaster Recovery Runbook

## Purpose

This page describes the order of operations after a major homelab outage.

Examples:

- prolonged power loss
- Proxmox node failure
- NAS outage
- router failure
- corrupted VM
- accidental deletion
- thin-pool exhaustion

## Priority 1: Network

Confirm:

1. UDM Pro powered and healthy
2. core switches online
3. DHCP working
4. trusted LAN reachable
5. Pi-hole reachable
6. internet available

Useful tests:

```bash
ping <gateway>
ping <pihole>
nslookup example.com <pihole>
```

## Priority 2: Storage

Confirm Synology NAS:

- powered on
- array healthy
- volumes mounted
- SMB available
- backup share available

Do not start dependent services aggressively if the NAS is unavailable and they expect mounts.

## Priority 3: Proxmox

Check cluster nodes:

```bash
pvecm status
pvesm status
```

Confirm:

- quorum
- storage availability
- thin-pool health
- critical VMs

## Priority 4: DNS and Reverse Proxy

Confirm Pi-hole.

Then confirm Traefik.

Test:

```bash
nslookup <service>
curl -vk https://<service>
```

## Priority 5: Critical Services

Recover in approximately this order:

1. Home Assistant
2. Vaultwarden
3. Authelia
4. core Docker hosts
5. Immich
6. media stack
7. secondary services

## VM Failure

If one VM is damaged:

1. identify latest known-good backup
2. confirm data disks / NAS data are intact
3. restore VM
4. verify network identity
5. verify DNS
6. verify mounts
7. start applications
8. test reverse proxy
9. test backup jobs

## NAS Failure

If NAS unavailable:

1. prevent services from writing to missing mount paths
2. identify which applications can safely remain stopped
3. restore NAS availability
4. confirm mount targets
5. remount shares
6. start dependent applications
7. verify backup integrity

## Thin-Pool Exhaustion

Do not reboot blindly.

Check:

```bash
lvs
pvesm status
```

Free guest data if possible, then:

```bash
sudo fstrim -av
```

Re-check host allocation.

## Documentation Availability

Because WikiMD lives on the NAS, consider maintaining a periodic offline export of critical
runbooks so documentation remains available during a NAS outage.

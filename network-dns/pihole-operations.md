# Pi-hole Operations

## Overview

Pi-hole is the primary internal DNS layer for the homelab.

It is used for:

- DNS filtering
- local records
- wildcard DNS
- service discovery support

## Docker Layout

Pi-hole has been run in Docker with persistent data under:

```text
/etc/pihole
/etc/dnsmasq.d
```

Exact host bind-mount locations should be documented on the service-specific page.

## Wildcard DNS

Local wildcard DNS has been implemented with dnsmasq configuration.

After changing dnsmasq files, Pi-hole FTL needs a reload / restart.

## Testing

Direct query:

```bash
nslookup <host> <pihole-ip>
```

or:

```bash
dig @<pihole-ip> <host>
```

## Reload / Restart

Preferred method depends on deployment.

In Docker:

```bash
docker restart pihole
```

If only config reload is required, use Pi-hole-supported reload mechanisms appropriate to the
version.

## Multiple Pi-hole Context

A separate Pi-hole instance has also been used for an isolated network.

Known custom port context:

```text
53535
```

Document which VLAN/network each instance serves.

## Failure Symptoms

If Pi-hole is down:

- internal hostnames may fail
- external DNS may fail depending on DHCP config
- Traefik services may appear offline by name
- direct IP access may still work

## Troubleshooting

```bash
docker logs pihole
dig @<pihole-ip> example.com
dig @<pihole-ip> <internal-host>
```

Check:

- container state
- upstream DNS
- port 53 binding
- local wildcard files
- firewall access from each VLAN

## Backup Scope

Back up:

- `/etc/pihole`
- `/etc/dnsmasq.d`
- custom local DNS files
- Docker Compose config

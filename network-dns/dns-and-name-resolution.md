# DNS and Name Resolution

## Overview

Internal DNS is provided primarily by Pi-hole.

The goal is for homelab services to be accessed by stable DNS names instead of IP addresses.

## Pi-hole

Pi-hole is used for:

- LAN DNS
- local records
- wildcard records
- filtering
- service routing support

## Local Wildcard Records

Wildcard records have been implemented with dnsmasq configuration.

Typical conceptual flow:

```text
*.local.example
    ->
Traefik IP
```

or:

```text
*.internal.example
    ->
internal reverse-proxy address
```

After editing dnsmasq configuration, reload / restart Pi-hole FTL.

## Home Assistant DNS Note

Home Assistant OS uses its own internal resolver path.

When troubleshooting a local hostname from Home Assistant, explicitly test against the
Pi-hole server:

```bash
nslookup hostname <pihole-ip>
```

Do not assume a failed resolution from the default HA shell resolver means Pi-hole itself
is misconfigured.

## Split-Horizon / Internal Resolution

Where a service is exposed publicly and internally, prefer internal clients resolving
directly to the LAN-side Traefik address rather than hairpinning through the public WAN path.

Benefits:

- lower latency
- less dependence on WAN state
- easier troubleshooting
- cleaner firewall behavior

## Diagnostic Commands

Query DNS:

```bash
nslookup <hostname>
dig <hostname>
dig @<pihole-ip> <hostname>
```

Query a specific record type:

```bash
dig A <hostname>
dig AAAA <hostname>
dig CNAME <hostname>
```

Trace resolver behavior:

```bash
dig +trace <hostname>
```

Check client resolver configuration:

```bash
cat /etc/resolv.conf
resolvectl status
```

## Failure Modes

### Service works by IP but not hostname

Check:

1. DNS record
2. wildcard record
3. client DNS server
4. Pi-hole FTL reload
5. local DNS cache
6. split-horizon target

### Hostname resolves but HTTPS fails

DNS is probably not the primary issue.

Check:

1. Traefik router
2. certificate
3. middleware
4. backend connectivity
5. firewall

### One VLAN resolves differently

Check:

- DHCP-provided DNS
- VLAN-specific firewall rules
- whether that network can reach Pi-hole
- whether multiple Pi-hole instances are in use

## Documentation Template

| Domain / Pattern | Resolves To | Purpose | Public? | Notes |
|---|---|---|---|---|
| `*.example` | | | | |
| `*.internal.example` | | | | |

# Reverse Proxy and Access Paths

## Purpose

This page documents the architecture around Traefik without replacing the existing
service-specific [Traefik](traefik) page.

## Reverse Proxy

Primary reverse proxy:

```text
Traefik v3
```

Traefik runs separately from the NAS.

Known responsibilities:

- TLS termination
- host routing
- middleware
- internal/public routing
- Authelia integration
- Cloudflare DNS challenge

## Public Request Path

```text
Internet
   |
   v
UDM Pro :443
   |
   v
Traefik
   |
   +-- router rule
   +-- TLS certificate
   +-- middleware
   |
   v
backend service
```

WAN TCP 443 is forwarded to Traefik.

## Internal Request Path

```text
LAN client
   |
   v
Pi-hole
   |
   v
internal Traefik address
   |
   v
service
```

Internal DNS should ideally keep LAN clients on the LAN path.

## Certificate Management

Cloudflare DNS challenge is used for certificate issuance.

Certificate state is stored in:

```text
acme.json
```

This file should be:

- backed up
- restricted by file permissions
- excluded from public repositories
- treated as sensitive

## Wildcard Certificates

Wildcard SANs have been used for internal/service subdomains.

Document each wildcard as:

| Wildcard | Used For | Resolver | Notes |
|---|---|---|---|
| `*.example` | | Cloudflare DNS | |
| `*.internal.example` | | Cloudflare DNS | |

## Backend Connectivity

Traefik must be able to reach the target service directly.

When a proxied service fails:

1. test backend from Traefik host
2. test router rule
3. check middleware
4. check certificate
5. inspect Traefik logs

Example backend test:

```bash
curl -v http://<backend>:<port>
```

HTTPS frontend test:

```bash
curl -vk https://<service-domain>
```

## Service Exposure Categories

Each service should be classified as one of:

### Public + application auth

Example class:

```text
public internet
 -> Traefik
 -> application login
```

### Public + Authelia

```text
public internet
 -> Traefik
 -> Authelia
 -> application
```

### Internal only

```text
LAN
 -> internal DNS
 -> Traefik
 -> application
```

### Direct LAN only

```text
LAN
 -> host:port
```

## Recommended Inventory

| Service | Domain | Backend | Exposure | Auth Layer | Notes |
|---|---|---|---|---|---|
| | | | Internal/Public | App/Authelia/Both | |

# Authelia Operations

## Overview

Authelia provides SSO / MFA protection for selected homelab services.

It is integrated with Traefik.

## Role

Authelia sits conceptually between Traefik and protected applications:

```text
client
  |
  v
Traefik
  |
  v
Authelia
  |
  v
application
```

## Use Cases

Authelia is useful for services that:

- lack strong authentication
- expose administrative interfaces
- are reachable from outside the trusted LAN
- benefit from centralized MFA

## Do Not Force It Everywhere

Some applications are poor candidates when:

- native mobile apps cannot complete proxy authentication
- WebSockets/API calls break
- application-native MFA is already strong
- service-to-service access needs direct API calls

## User Management

Authelia has a smaller user set than applications like Immich.

This is expected: Authelia users represent people allowed through the proxy layer, not every
account that exists inside every application.

## MFA

Authelia can provide a second authentication factor in front of selected services.

This is especially useful for infrastructure/admin interfaces.

## Traefik Integration

The exact middleware definition belongs in the existing Traefik/Authelia config pages.

Operationally, if a protected service fails:

1. test backend directly
2. test through Traefik without assuming Authelia
3. inspect Authelia logs
4. inspect middleware routing
5. verify cookies/domain scope
6. verify user exists

## Logs

Typical Docker investigation:

```bash
docker logs authelia
docker logs -f authelia
```

## Backup

Back up:

- configuration
- user database / identity files
- storage backend
- notification settings
- secret references

Do not put the actual secret values in WikiMD.

## Upgrade Caution

Authelia configuration keys have changed across versions.

One previous warning indicated legacy server keys such as:

```text
server.host
server.port
server.path
```

were deprecated in favor of:

```text
server.address
```

When upgrading, check logs for deprecation warnings before the next major release removes
compatibility mappings.

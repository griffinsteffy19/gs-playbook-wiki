# Vaultwarden Operations

## Overview

Vaultwarden runs in the homelab as a high-value password-management service.

It has previously been associated with the `record-book` Docker host.

## Criticality

Vaultwarden should be treated as one of the highest-priority applications for:

- backup
- restore testing
- TLS
- secrets management
- access control

## Backup Scope

Back up:

- Vaultwarden data directory
- database
- attachments
- configuration
- Compose file
- environment configuration
- any required encryption / admin secret references

## Restore Test

A proper restore test should verify:

1. service starts
2. users can log in
3. vault data is present
4. attachments open
5. reverse proxy works
6. TLS works
7. admin interface behavior is correct

## Reverse Proxy

Vaultwarden should be accessed only over HTTPS.

If using Traefik, ensure:

- correct router
- correct backend port
- WebSocket compatibility where required
- no accidental plaintext WAN exposure

## Client Troubleshooting

A prior Alfred integration issue involved:

```text
client_secret
```

If third-party clients fail while the web UI works:

- verify the client supports the current auth flow
- verify required API endpoints
- check whether proxy middleware interferes
- test direct application behavior if safe

## Backup Priority

Recommended:

```text
Tier: Critical
RPO: low
Restore testing: regular
```

## Security Notes

Do not document:

- admin token
- SMTP passwords
- database passwords
- private keys

Document only where those values live.

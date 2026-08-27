# Jellyfin Operations

## Overview

Jellyfin has been deployed as an alternate/self-hosted media server.

It is routed through Traefik.

## Reverse Proxy

Confirm:

- router hostname
- backend port
- WebSocket support if needed
- TLS
- middleware

## Media Storage

Jellyfin consumes media from Synology-backed mounts.

Before troubleshooting Jellyfin library failures, verify host mount state.

## Collections and Tags

Collection/tag propagation has been used to support parental-control workflows.

A useful pattern is:

```text
collection
   ->
apply tag
   ->
use tag in access/filtering logic
```

This is preferable to manually tagging every item one by one.

## Bulk Tagging

When automating collection -> tag propagation:

1. enumerate collection members
2. inspect existing tags
3. add missing target tag
4. preserve unrelated tags
5. test on one collection first

## Backup

Back up:

- Jellyfin config
- metadata if important
- user accounts
- plugin config

Bulk media files remain on NAS storage.

## Troubleshooting

### UI works but media unavailable

Check:

- NAS mounts
- permissions
- library paths

### Reverse proxy fails

Test direct backend:

```bash
curl -v http://<jellyfin-host>:<port>
```

Then test frontend:

```bash
curl -vk https://<jellyfin-domain>
```

# Kometa Operations

## Overview

Kometa manages Plex metadata and collections.

Known configuration path:

```text
/opt/data/kometa/config
```

## Known Collection Work

Previously discussed:

- James Bond
- Star Wars
- MCU
- Pixar
- Disney

Some collections worked while others required list/source troubleshooting.

## Radarr Integration Preference

Desired behavior:

```yaml
add_missing: true
search: false
```

This means:

- add missing titles to Radarr
- do not automatically launch searches

This keeps Kometa from unexpectedly starting large download batches.

## Collection Ordering

Desired ordering has included:

- timeline order
- release order

depending on collection.

## Troubleshooting Missing Collections

Check:

1. metadata source
2. list availability
3. Plex library target
4. YAML indentation
5. collection definition
6. API key / connectivity
7. Kometa logs

## Logs

Typical run:

```bash
docker logs kometa
```

or inspect Kometa's own generated logs under its config/log directory.

## Backup Scope

Back up:

- `/opt/data/kometa/config`
- custom YAML
- overlays
- scripts
- secrets references

Generated Plex metadata itself is not the primary backup target.

## Safe Test Pattern

Before applying a new collection globally:

1. run only one library
2. run one collection
3. disable destructive changes
4. inspect output
5. enable add-missing only after verification

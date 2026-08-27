# Wiki (Wikmd)

This wiki. Documented here so future-you can redeploy or troubleshoot it
without rediscovery.

## Where it lives

- **Host:** Synology NAS (`longsnapper.gs`)
- **Container:** `linbreux/wikmd:latest`
- **Port:** `9454` (host) → `5000` (container)
- **Content directory (host):** `/volume1/wiki/wikimd`

## Compose file

```yaml
services:
  wikmd:
    image: linbreux/wikmd:latest
    container_name: wikmd
    environment:
      - PUID=1026
      - PGID=100
      - TZ=America/Indianapolis
    volumes:
      - /volume1/wiki/wikimd:/wiki:rw
    ports:
      - 9454:5000
    restart: unless-stopped
```

## Known gotchas

- **Volume mount path matters exactly.** Wikmd expects `homepage.md` and
  friends directly inside the container's `/wiki` — if the host-side
  mount points at a parent directory instead of where the actual content
  files live, Wikmd starts cleanly but errors on every page load with
  `No such file or directory`. Confirm with `docker exec wikmd ls /wiki`.
- **`homepage.md` is required** — it's the landing page Wikmd looks for
  by convention; a fresh instance with no pages will error until it exists.
- **Internal git repo**: Wikmd auto-initializes its own git repo inside
  the wiki content directory to track page edits (visible in container
  logs on first start: `Initializing existing repo >>> wiki`). This is
  separate from any of your own project repos — don't `git push` from
  inside it expecting it to go to GitHub unless you've deliberately added
  a remote (`git remote add origin ...`).
- **Link syntax**: plain Markdown, linking to the page's filename slug
  (no extension) — `[Display Text](page-slug)`. Not `[[wikilinks]]`.

## Backup

This wiki's content isn't currently included in the [Service Backup](service-backup)
rollout — it lives on the NAS itself, not a node's `/opt/data`. Worth
deciding whether to add `/volume1/wiki/wikimd` as its own backup target
or rely on Synology's own snapshot/backup tools for NAS-local data.
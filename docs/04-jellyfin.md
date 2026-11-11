# 04 – Jellyfin

Local media server for the family. `http://192.168.0.252:8096`

## Deployment

Defined in [docker-compose.yml](../docker/docker-compose.yml). Runs as UID/GID 1000. Media mounted **read-only** from `/mnt/media` → `/media` in the container.

## Initial setup

- Server name: display-only, changeable in Dashboard → General.
- Libraries:

| Library | Type | Path |
|---|---|---|
| Movies | Movies | `/media/Movies` |
| Shows | Shows | `/media/Shows` |

- Separate user account per family member (Dashboard → Users).
- Clients: Jellyfin apps on TVs/phones pointed at `http://192.168.0.252:8096`.

## Security settings

| Setting                       | Value              | Why                                                 |
| ----------------------------- | ------------------ | --------------------------------------------------- |
| Allow remote connections      | **Off**            | Local-only for now; turn on when Tailscale is added |
| Automatic port mapping (UPnP) | **Off**            | Prevents Jellyfin opening a router port on its own  |

## Media naming

Jellyfin identifies media from folder and file names.

```
Movies/Inception (2010)/Inception (2010).mkv
Shows/Breaking Bad (2008)/Season 01/Breaking Bad S01E01.mkv
```

- Fix mismatches: item → ⋯ → **Identify**. Force a match with `[tmdbid-27205]` in the folder name.
- Bulk renaming: tinyMediaManager.
- Copying files in from Windows: WinSCP (SFTP to `192.168.0.252`).

## Content sources

None yet. For now it is used for testing purposes. Eventually will add family videos or other owned media.
## Hardware transcoding (future)

i5-10400F has no iGPU (no Quick Sync). Future plan: pass through the GTX 1650 Super and enable **NVENC** ([docs](https://jellyfin.org/docs/general/post-install/transcoding/hardware-acceleration/nvidia)).
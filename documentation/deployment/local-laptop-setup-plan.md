# Local Laptop Setup Plan (Windows, offline listening)

**Status:** ⏳ Plan only | Not implemented, nothing in the repo depends on it

## Overview
Run RMAB + Audiobookshelf + Prowlarr + qBittorrent in Docker on a Windows laptop, request books in RMAB, then copy finished files to one phone for offline listening. Laptop can be off once files are on the phone.

## Key Details
- **Goal:** single user, single phone, offline playback. No multi-device sync, no remote streaming.
- **RMAB is not a player.** It fetches and files books; a media server and a phone player are separate.
- **Media server is required:** setup wizard step 3 needs Plex or Audiobookshelf ([../setup-wizard.md](../setup-wizard.md)). Plan uses Audiobookshelf.
- **GPU/VRAM:** not used. Needs RAM (~2-4 GB for the stack), any dual-core CPU, disk (~200 MB-1 GB per book).
- **Arch:** published image is `linux/amd64` + `linux/arm64` (`.github/workflows/build-unified-image.yml`).
- **Target machine (checked):** Windows 11, 7.2 GB RAM, D: 93 GB free, winget present, WSL not installed, no Docker.

## Components
| Piece | Role | Default port (confirm at setup) |
|---|---|---|
| RMAB (`ghcr.io/kikootwo/readmeabook:latest`) | Request UI, search, rank, organize | 3030 |
| Audiobookshelf | Library + media server RMAB files into | 13378 |
| Prowlarr | Indexer search; yields API key for RMAB | 9696 |
| qBittorrent (or SABnzbd) | Downloader | 8080 |
| Phone player | Plays copied files (Android: Smart AudioBook Player; iOS: Prologue/BookPlayer) | n/a |

RMAB image bundles Postgres + Redis ([unified.md](unified.md)). RMAB volumes: `/app/config`, `/downloads`, `/media`.

## Steps
1. **Docker (user, needs admin):** Admin PowerShell → `wsl --install --no-distribution` → restart → `winget install -e --id Docker.DockerDesktop` → start Docker Desktop, wait for "Engine running". No Docker account needed.
2. **RAM cap:** set `memory=3GB` in `%UserProfile%\.wslconfig`, then `wsl --shutdown`.
3. **Folders:** `D:\abs\{audiobooks,downloads,config}` plus per-service config dirs.
4. **Compose file** in `D:\abs\` (outside the repo): the four services above.
5. **Paths must agree:** RMAB, downloader and Audiobookshelf must see the same download/media files; mismatch = "RMAB can't find my downloaded files" ([volume-mapping.md](volume-mapping.md)).
6. **First-run logins (local only, no external signup):** Audiobookshelf admin; RMAB admin via wizard; qBittorrent login; Prowlarr auth + copy API key.
7. **RMAB wizard:** admin → Audiobookshelf → Prowlarr (key + indexers) → qBittorrent → paths → finish.
8. **Use:** request a book → wait for "available" → find `Author/Title` (usually one `.m4b`) in `D:\abs\audiobooks` → copy to phone (USB/cloud) → play offline.

## Role split
- **Claude:** write compose + folders, check Docker, start containers, read logs, fix path issues.
- **User:** install Docker Desktop (admin/UAC + restart), choose passwords, add indexers they are entitled to use, click through web UIs, copy files to phone.

## Alternatives
- **Skip RMAB:** Audiobookshelf, qBittorrent, Prowlarr have native Windows installers; search and file books by hand. No Docker needed.
- **Native RMAB:** needs Node, Postgres, Redis (no official Windows build) and ffmpeg; unsupported.
- **Free VPS:** possible (image is arm64-capable) but most free tiers forbid torrenting and have small disks; hybrid = download at home, upload to VPS.
- **Multi-device sync/streaming:** Audiobookshelf feature; needs the server on, plus Tailscale for use away from home.

## Open Decisions
- Torrents (qBittorrent) vs Usenet (SABnzbd; paid provider)
- Phone OS (player app)
- Install location (default `D:\abs\`)

## Notes
- Automated sourcing uses torrent/Usenet indexers; use only for content you have the right to obtain.
- BookDate (AI picks) is optional and needs an external API key; skip it.

## Related: [unified.md](unified.md), [docker.md](docker.md), [volume-mapping.md](volume-mapping.md), [../setup-wizard.md](../setup-wizard.md)

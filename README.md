# PiNAS App Catalog

App catalog for [PiNAS](https://github.com/your-repo/pinas) — a modern NAS OS for Raspberry Pi 5.

## Overview

This catalog provides 28 ready-to-install applications for PiNAS. Apps are installed through the PiNAS App Center UI and run as Docker containers on LibreELEC.

## Apps (28)

### Containers
| App | Description |
|-----|-------------|
| **Docker** | Container runtime (binary install, no Docker dependency) |
| **Portainer** | Web-based Docker management UI |

### Media & Downloads
| App | Description |
|-----|-------------|
| **Plex** | Media server (network_mode: host) |
| **Jellyfin** | Open-source media server (network_mode: host) |
| **Emby** | Media server |
| **Sonarr** | TV show download manager |
| **Radarr** | Movie download manager |
| **Lidarr** | Music download manager |
| **qBittorrent** | BitTorrent client |
| **Transmission** | Lightweight BitTorrent client |
| **SABnzbd** | Usenet download manager |
| **PhotoPrism** | AI-powered photo management (Compose: app + MariaDB) |

### Network
| App | Description |
|-----|-------------|
| **Samba** | Windows file sharing (SMB/CIFS, binary install) |
| **Pi-hole** | Network-wide ad blocker (network_mode: host, cap_add: NET_ADMIN) |
| **AdGuard Home** | DNS ad blocker |
| **WireGuard** | VPN server |
| **Nginx Proxy Manager** | Reverse proxy with SSL |

### Utilities
| App | Description |
|-----|-------------|
| **Nextcloud** | Cloud storage & collaboration (Compose: app + PostgreSQL + Redis) |
| **Home Assistant** | Home automation (network_mode: host, privileged) |
| **Syncthing** | File synchronization |
| **Vaultwarden** | Password manager (Bitwarden-compatible) |
| **Grafana** | Monitoring dashboards |
| **Uptime Kuma** | Service uptime monitoring |
| **File Browser** | Web file manager |
| **Code Server** | VS Code in the browser |
| **Node-RED** | Flow-based automation |
| **Paperless-ngx** | Document management (Compose: app + PostgreSQL + Redis) |
| **Duplicati** | Backup solution |

## Structure

```
app-catalog/
├── catalog.json              # Index with metadata for all apps
└── apps/
    ├── docker/manifest.json
    ├── portainer/manifest.json
    ├── jellyfin/manifest.json
    ├── nextcloud/manifest.json
    └── ...                   # 28 app manifests
```

## Manifest Format

Each app has a `manifest.json` with:

```json
{
  "id": "app-id",
  "name": "App Name",
  "version": "1.0.0",
  "description": { "en": "...", "fr": "..." },
  "author": "...",
  "requirements": {
    "min_ram": 512,
    "min_disk": 100,
    "arch": ["aarch64", "x86_64"],
    "dependencies": ["docker"]
  },
  "install": {
    "type": "docker",
    "steps": [...]
  },
  "uninstall": { "steps": [...] },
  "frontend": {
    "component": "IframeApp|WebviewApp|ServiceApp",
    "icon": "mdi:application",
    "gradient": "from-blue-500 to-purple-500",
    "config": { "port": 8080 },
    "i18n": { "en": {...}, "fr": {...} }
  }
}
```

### Install Step Types

| Step | Description |
|------|-------------|
| `download` | Download file (optional SHA256 verification) |
| `extract` | Extract tar.gz archive |
| `mkdir` | Create directory |
| `copy`, `symlink`, `chmod` | File operations |
| `template`, `write_file` | Write config files (base64) |
| `exec` | Run shell command |
| `docker_pull` | Pull Docker image |
| `docker_create` | Create container (with full ContainerConfig) |
| `docker_start` / `docker_stop` / `docker_rm` | Container lifecycle |
| `compose_up` | Start multi-container app via Docker Compose |
| `compose_down` | Stop and remove Compose app |

### Container Config

`docker_create` supports:
- `image`, `ports`, `volumes`, `environment`
- `network` (host, bridge)
- `restart_policy` (always, unless-stopped)
- `cap_add` / `cap_drop` (NET_ADMIN, SYS_ADMIN, etc.)
- `command`, `entrypoint`, `user`
- `dns`, `extra_hosts`, `tmpfs`
- `devices` (/dev/dri for GPU)
- `privileged` mode

### Variables

| Variable | Value |
|----------|-------|
| `${PACKAGES_DIR}` | `/storage/.pinas/packages` |
| `${DATA_DIR}` | `/storage/.pinas/data` |
| `${APP_DATA_DIR}` | `/storage/.pinas/data/apps/{app_id}` |
| `${BIN_DIR}` | `/storage/.pinas/bin` |
| `${DOWNLOADS_DIR}` | `/storage/.pinas/downloads` |
| `${ARCH}` | `aarch64` or `x86_64` |
| `${DEVICE_HOSTNAME}` | System hostname |

## Conversion from Umbrel

Apps were converted from [Umbrel App Store](https://github.com/getumbrel/umbrel-apps) using `scripts/convert-umbrel.py`:

```bash
# Convert a single app
python3 scripts/convert-umbrel.py umbrel-apps/jellyfin -o app-catalog/apps/jellyfin

# Batch convert
./scripts/convert-umbrel-batch.sh
```

The script handles:
- Single-service apps → `docker_pull` + `docker_create` + `docker_start` steps
- Multi-service apps → `compose_up` step with inline YAML
- Variable substitution (Umbrel `${APP_DATA_DIR}` → PiNAS equivalent)
- Filtering out Umbrel-specific services (`app_proxy`)

## Roadmap

- [x] Docker & Portainer (binary + Docker install)
- [x] Samba (binary install)
- [x] 25 Umbrel apps converted (media, network, utilities)
- [x] Docker Compose support for multi-container apps
- [x] French translations for all apps
- [ ] Add more apps (Gitea, Immich, Kavita, Calibre-Web)
- [ ] Auto-update manifests from upstream
- [ ] App screenshots in catalog UI

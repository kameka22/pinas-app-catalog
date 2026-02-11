# App Catalog PiNAS

Hébergé sur GitHub : `kameka22/pinas-app-catalog`

## Structure

```
app-catalog/
├── catalog.json              # Index avec metadata (28 apps)
└── apps/
    ├── docker/manifest.json  # Manifest Docker (binaire)
    ├── portainer/manifest.json
    ├── samba/manifest.json
    ├── jellyfin/manifest.json
    ├── nextcloud/manifest.json  # Multi-container (Compose)
    └── ...
```

## Format Manifest

```json
{
  "id": "app-id",
  "name": "App Name",
  "version": "1.0.0",
  "description": { "en": "...", "fr": "..." },
  "author": "...",
  "license": "MIT",
  "requirements": {
    "min_ram": 512,
    "min_disk": 100,
    "arch": ["aarch64", "x86_64"],
    "dependencies": ["docker"]
  },
  "install": {
    "type": "binary|docker",
    "steps": [
      { "action": "download", "url": "...", "dest": "...", "sha256": "..." },
      { "action": "extract", "src": "...", "dest": "..." },
      { "action": "mkdir", "path": "..." },
      { "action": "symlink", "src": "...", "dest": "..." },
      { "action": "chmod", "path": "...", "mode": "755" },
      { "action": "template", "src": "...", "dest": "..." },
      { "action": "exec", "command": "..." }
    ]
  },
  "uninstall": {
    "steps": [
      { "action": "exec", "command": "...", "ignore_error": true },
      { "action": "delete", "path": "..." }
    ]
  },
  "files": {
    "template-name": "base64-encoded-content"
  },
  "frontend": {
    "component": "IframeApp|WebviewApp|ServiceApp|DockerApp",
    "icon": "mdi:application",
    "gradient": "from-blue-500 to-purple-500",
    "window": { "width": 1200, "height": 800 },
    "config": { "url": "http://localhost:9000" },
    "i18n": { "en": {}, "fr": {} }
  }
}
```

## Types d'installation

| Type | Usage |
|------|-------|
| `binary` | Télécharge et extrait des binaires statiques (ex: Docker) |
| `docker` | Docker pull/create/start des containers |
| `compose` | Docker Compose pour apps multi-container (ex: Nextcloud) |

## Variables substituées

| Variable | Valeur |
|----------|--------|
| `${PACKAGES_DIR}` | `/storage/.pinas/packages` |
| `${DATA_DIR}` | `/storage/.pinas/data` |
| `${APP_DATA_DIR}` | `/storage/.pinas/data/apps/{app_id}` |
| `${BIN_DIR}` | `/storage/.pinas/bin` |
| `${DOWNLOADS_DIR}` | `/storage/.pinas/downloads` |
| `${ARCH}` | `aarch64` ou `x86_64` |
| `${DEVICE_HOSTNAME}` | Hostname système |

## SHA256

Vérification optionnelle. Par architecture :
```json
{ "sha256_aarch64": "abc123...", "sha256_x86_64": "def456..." }
```

## Gestion des dépendances

- Définies dans `requirements.dependencies[]`
- Le frontend bloque l'installation si dépendances manquantes
- Fonctions : `isPackageInstalled(id)`, `getMissingDependencies(deps)`, `canInstall(app)`

## Configuration frontend des apps

`frontend.config` est transmise via `appConfig` dans le store `desktop.ts` et passée au composant lors de l'ouverture de la fenêtre.

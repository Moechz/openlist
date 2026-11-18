# openlist-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [OpenList](https://github.com/OpenListTeam/OpenList).

## What this is

A four-file TOS Docker application archive (`openlistdocker.tar.gz`) that installs
OpenList on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id `openlistdocker`)
- `openlistdocker.lang` — 23-language superset (TOS requires 14)
- `openlistdocker.svg` — application icon (370 bytes, pure vector)
- `docker-compose.yml` — one service, image `openlistteam/openlist:v4.2.9`

## How it works

| | |
|---|---|
| Container image | `openlistteam/openlist:v4.2.9` (Docker Hub, used unmodified) |
| Open | `http://<NAS-IP>:8086/` (container port 5244) |
| Runs as | non-root, `user: "1000:1000"` |
| Persistent data | `/opt/openlist/data` → `/Volume1/DockerAppData/openlistdocker/data` |
| Logs | `docker logs openlistdocker` (stdout/stderr) |
| Health check | `GET /ping` on container port 5244 |

## First start

Open `http://<NAS-IP>:8086/`. The first visit shows OpenList's **setup wizard**; create the
administrator account there. Nothing is written to the container log, and the package sets no
`OPENLIST_ADMIN_PASSWORD` — so no password ever ends up in `docker logs`.

If the administrator password is lost:

```bash
docker exec openlistdocker ./openlist admin set <new-password>
```

Upstream requires the data folder to be writable by the container user; the TOS App Center
creates and chowns `/Volume1/DockerAppData/openlistdocker/` for uid 1000. If the entrypoint
reports “Current user does not have write and/or execute permissions for the ./data
directory”, the ownership of that folder is wrong — fix it with
`chown -R 1000:1000 /Volume1/DockerAppData/openlistdocker/` and restart the container.

## Third-party services (API disclosure)

The developer operates no server in this path and receives no data. The application contacts
only the services **you configure**, plus the Docker registry at install time.

| Service | Endpoint | Data sent | Destination / region |
|---|---|---|---|
| A storage you mount | that provider's official API/SDK endpoint, for example `graph.microsoft.com`, `www.googleapis.com`, `api.aliyundrive.com`, `pan.baidu.com`, an S3 endpoint, or the WebDAV/SFTP/FTP/SMB address you enter | the stored credentials/tokens and the file operations you request (list, read, upload, move) | the provider you chose, or your own server |
| OAuth sign-in for such a drive, only when you add it | the provider's authorisation endpoint, for example `login.microsoftonline.com` or `accounts.google.com` | the authorisation request; the token returned is stored in the app database | the provider's servers |
| WebDAV / API clients you point at the app | the address you give them | the file operations you perform | your LAN |
| Image pull at install | `registry-1.docker.io` or your registry mirror | anonymous pull | Docker Hub, US / your mirror |

There is no telemetry, no analytics, no advertising SDK and no update check in the
application. See `PRIVACY.md` §6 for the same table with more detail.

## Privacy

The application runs entirely on your NAS and the developer has no access to its data. The
full policy — retention period, user rights (access/export, correction, deletion), security
measures, third-party sharing and the deletion channel — is in [PRIVACY.md](PRIVACY.md), and
the package `help` field opens it directly from the App Center entry:
<https://github.com/Moechz/openlist/blob/main/PRIVACY.md>.

| Topic | Detail |
|---|---|
| Data processed | App database (users, mounts and their credentials, index cache), settings, logs on stdout |
| Where | `/Volume1/DockerAppData/openlistdocker/data` on your NAS |
| Retention | Until **you** delete it; no automatic expiry, no upload |
| Rights | Copy the data folder (export), edit in the web UI (correct), uninstall with “delete data” or remove the folder (delete) |
| Deletion channel | The data folder, or moechz@163.com (answered within 30 days) |

## Licence and attributions

| Component | Licence | Notes |
|---|---|---|
| OpenList | AGPL-3.0 | upstream application, used unmodified inside the image |
| Container image `openlistteam/openlist:v4.2.9` | see the image's own metadata | pulled from Docker Hub at install time |
| This packaging | MIT (see `LICENSE`) | compose file, metadata, translations, icon |

“OpenList”, “TerraMaster” and “TOS” are trademarks of their respective owners. This is an
independent community package by Moechz and is not affiliated with the upstream project or
TerraMaster. Full details: [NOTICE.md](NOTICE.md).

## Design highlights

- **Non-root.** `user: "1000:1000"`; the image itself also declares a non-root user
  (`openlist`, uid 1001). No `privileged`, no `network_mode: host`, no Docker socket, no
  `cap_add`.
- **Fixed image tag, never `:latest`**, Docker Hub only.
- **Health check** probing `/ping`, a static route of the application that answers before the
  setup wizard is completed.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip without a
  timestamp, plus a review-standards self-check.

## Runtime file manifest

Everything the application writes on the NAS lives inside its own data directory; the
container-internal `/tmp` is discarded with the container. The table lists what is created and
how it behaves. Nothing else is written.

| Path | Purpose | Created when | Growth bound | Lifecycle |
|---|---|---|---|---|
| `/Volume1/DockerAppData/openlistdocker/data` | OpenList application data (database, mounts, index cache, everything the container writes outside /tmp) | on first start / on demand | bounded by your own mounts | persistent; kept on uninstall unless you ask to delete it |

## Architectures

The TOS store takes one package per architecture. This release provides both, built from the
same sources by `scripts/build-dual.sh`:

| Architecture | Package | Size | SHA-256 |
|---|---|---|---|
| x86_64 | `openlistdocker_x86_64.tar.gz` | 4298 B | `490e38bf25667ab35a62163f4822e6ffc49d94ae20a122bc83cde59b67fd5e6f` |
| aarch64 | `openlistdocker_aarch64.tar.gz` | 4298 B | `967ed252f3fb715e3480f4b14a470dbda17f54acfd1082a05e258cb45239cc15` |

Both packages pass the official `TOSAppSelfTestingTool -c` check and the package's own
`tools/verify.py`. The on-device E2E was run for `4.2.2-2`; the `4.2.9` image bump is pending a
fresh on-device run (Docker applications have to be installed with the official tool's `-i`
mode, not through the App Center install page).

## Build

```bash
scripts/build-dual.sh 4.2.9-1     # both architectures + official check
scripts/build.sh 4.2.9-1 x86_64   # one architecture
scripts/build.sh                  # current version from tosapp.json
```

The build stages the four files, substitutes the version/platform placeholders, removes
CRLF/BOM/AppleDouble, then runs `tools/verify.py`, a review-standards self-check (archive
layout, JSON validity, required fields, 14+ language coverage, version consistency,
Docker-Hub-only images, fixed tag, reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data path below the application data root, never the `/Volume*`
wildcard, comment-free sequence blocks, `x-app-meta` position, no literal secrets and the SVG
size/element limits).

## Local verification

```bash
./TOSAppSelfTestingTool -c out/openlistdocker_x86_64.tar.gz   # official spec check
./TOSAppSelfTestingTool -i out/openlistdocker_x86_64.tar.gz   # install on this machine
bash test/install-and-verify.sh                               # on-device E2E (needs root)
```

> Docker applications cannot be side-loaded through the App Center's manual install page (it
> accepts `.deb` only); the official self-testing tool's `-i` mode or `docker compose` are the
> only ways to install one for testing.

## Store submission checklist

1. Public GitHub repo (code + README only; internal docs and build products are ignored).
2. Release **tag `v4.2.9-1`** with four assets: `openlistdocker_x86_64.tar.gz` + `.sha256` and
   `openlistdocker_aarch64.tar.gz` + `.sha256` (one archive per architecture, architecture
   suffix in the file name).
3. Developer platform: one App record per architecture (`x86_64`, `aarch64`), package type
   Docker, repository URL, version `4.2.9-1`.
4. When resubmitting after a review, put the item-by-item reply in the submission remarks.

## Limitations

- No storage is configured out of the box; add your local folders or cloud drives in the web
  UI under “Storage”.
- The web UI is plain HTTP on the LAN. Put it behind a TLS reverse proxy before exposing it to
  the internet.
- Docker applications cannot be upgraded in place in the App Center: updating means
  uninstalling and reinstalling, which keeps the data folder and lets OpenList migrate its
  database on start.
- Cloud-drive features depend on the provider's own API and can break when that provider
  changes it; the upstream project releases fixes, and this package is rebuilt for them.

## Support

- Issues and questions: <https://github.com/Moechz/openlist/issues>
- Email: moechz@163.com
- Privacy: [PRIVACY.md](PRIVACY.md) · Licence: [NOTICE.md](NOTICE.md)

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.

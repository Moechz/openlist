# Privacy Policy — OpenList for TerraMaster TOS (`openlistdocker`)

**Last updated:** 18 November 2026
**Data controller / contact:** Moechz — moechz@163.com

This policy covers the TOS application `openlistdocker` (the “app”) and its packaging. The
app is a thin wrapper: it runs the official `openlistteam/openlist` container image,
unmodified, on your own NAS.

## 1. Summary

- The app runs **entirely on your NAS**. The developer does not receive, collect or have
  access to your data, and operates no server that the app talks to.
- No telemetry, no analytics, no advertising, no account, no cloud service operated by the
  developer.
- The app contacts only the storage providers and clients **you configure**, plus the Docker
  image registry at install time.
- The application performs no version check of its own.

## 2. What data is processed

| Container path | Stored on the NAS at |
|---|---|
| `/opt/openlist/data` | `/Volume1/DockerAppData/openlistdocker/data` |

That folder holds the application's own state, written by OpenList:

- the user accounts of the app (username, salted password hash, permissions, tokens);
- the storage mounts you add, including the credentials, cookies or refresh tokens the
  provider needs, so the app can reach that drive again after a restart;
- the storage index and metadata cache (file and folder names, sizes, timestamps);
- settings, task records and the SQLite database;
- logs, which OpenList writes to stdout/stderr — they are visible through
  `docker logs openlistdocker` and are not kept in a file inside the data folder.

**Your files stay where they are.** OpenList lists and streams files from the local folders
and cloud drives you mount; it does not copy them into the data folder. The only exception is
an offline-download task that **you** start, whose result is written to the storage target you
choose for that task.

## 3. Retention period

- **Application data:** kept until **you** delete it. There is no automatic expiry and no
  upload. Uninstalling the app keeps the folder unless you tick “delete data”.
- **Logs:** live in the container's stdout/stderr; Docker rotates them according to the log
  driver configured on your NAS. Stopping and recreating the container discards them.
- **Cache / index:** rebuilt from the mounted storages on demand; safe to delete at any time
  (the storages themselves are not touched).

## 4. Your rights and the deletion channel

| Right | How |
|---|---|
| Access / export | Copy the folder `/Volume1/DockerAppData/openlistdocker/data` — it is the complete state of the app. |
| Correct | Change users, mounts and settings in the app's own web UI. |
| Delete | Uninstall the app with “delete data”, or stop the container and remove `/Volume1/DockerAppData/openlistdocker/`. |
| Assistance | Email **moechz@163.com** with the subject “OpenList data request”; a reply follows within 30 days. |

The developer holds no copy of your data, so deletion is normally completed by removing the
folder on your own NAS. The e-mail channel exists for questions and for requests about the
website or the publishing records.

## 5. Security measures

- The container runs as **uid 1000**, never root; no `privileged`, no `cap_add`, no host
  network, no Docker socket, no volumes beyond its own data folder.
- Only one port is published (`8086` on the NAS → `5244` in the container).
- The image tag is **pinned** (`openlistteam/openlist:v4.2.9`); the package is rebuilt when
  upstream ships fixes.
- The data folder is owned by the application user; the files inside are not world-readable
  beyond what the packaging user can read on the NAS.
- The web UI is served over **plain HTTP on your LAN**. Put it behind a TLS reverse proxy
  before exposing it to the internet, and set a strong administrator password in the setup
  wizard on first start.

The storage credentials stored in the app's database are only as safe as your NAS: anyone who
can read `/Volume1/DockerAppData/openlistdocker/` can read them. Keep that folder out of
shared folders and backups you do not control.

## 6. Third-party sharing and API disclosure

The developer shares nothing and receives nothing. The app contacts only the endpoints below,
and only for the storages and features **you** configure.

| Service | Endpoint | Data sent | Destination / region |
|---|---|---|---|
| A storage you mount | that provider's official API or SDK endpoint (for example `graph.microsoft.com`, `www.googleapis.com`, `api.aliyundrive.com`, `pan.baidu.com`, an S3 endpoint, or the WebDAV/SFTP/FTP/SMB address you enter) | the credentials/tokens you stored and the file operations you request (list, read, upload, move) | the provider you chose, or your own server |
| OAuth sign-in for such a drive, only when you add it | the provider's authorisation endpoint (for example `login.microsoftonline.com` or `accounts.google.com`) | the authorisation request; the token returned is stored in the app database | the provider's servers |
| WebDAV / API clients you point at the app | the address you give them | the file operations you perform | your LAN |
| Image pull at install | `registry-1.docker.io` or your registry mirror | anonymous pull | Docker Hub, US / your mirror |

No data is sold or shared with advertisers. There is no third-party analytics or tracking SDK
in the app, and the upstream project states that it collects no personal data (see
<https://doc.oplist.org/privacy>).

## 7. Children

The app is a self-hosted file-management utility and is not directed at children.

## 8. Changes

Material changes are published in this file and reflected in the “Last updated” date.

## 9. Contact

- Moechz — **moechz@163.com**
- Package repository and issues: <https://github.com/Moechz/openlist>
- Website privacy page: <https://www.moechz.com/privacy.html>
- Upstream project privacy statement: <https://doc.oplist.org/privacy>

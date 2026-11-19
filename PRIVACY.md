# Privacy Policy — OpenList for TerraMaster TOS (`openlistdocker`)

**Last updated:** 19 November 2026
**Data controller / contact:** Moechz — moechz@163.com

This policy covers the TOS application `openlistdocker` (the “app”) and its packaging. The
app is a thin wrapper: it runs the official `openlistteam/openlist` container image,
unmodified, on your own NAS.

## 1. Summary

- The app runs **entirely on your NAS**. The developer does not receive, collect or have
  access to your data, and operates no server that the app talks to.
- No telemetry, no analytics, no advertising and no account with the developer.
- The app talks to the **storage providers you configure**, to the **Docker image registry**
  at install time, and — only when you use the corresponding feature — to the third-party
  preview, OCR and single-sign-on services listed in §6. The upstream project's CDN also
  serves a few static web assets.
- Every one of those endpoints is listed in §6 with the data it receives. The developer
  operates none of them and receives nothing.
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
choose for that task. Preview and OCR features described in §6 work on the file you open — the
Office/Google preview services fetch it from the link the app generates, and OCR uploads the
image or PDF you submit to the OCR endpoint.

## 3. Retention period

- **Application data:** kept until **you** delete it. There is no automatic expiry and no
  upload. Uninstalling the app keeps the folder unless you tick “delete data”.
- **Logs:** live in the container's stdout/stderr; Docker rotates them according to the log
  driver configured on your NAS. Stopping and recreating the container discards them.
- **Cache / index:** rebuilt from the mounted storages on demand; safe to delete at any time
  (the storages themselves are not touched).
- **Preview links:** the share link the app generates follows OpenList's link-expiry
  setting (default `0` = no expiry; set a value to make preview links expire). The
  third-party preview service does not keep a copy beyond what its own policy states.

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
- Preview, OCR and SSO are optional. If you do not want a service from §6 to receive
  anything, do not open that preview, do not run OCR, and do not enable that sign-in method;
  the defaults can be changed in Settings (preview / OCR API / SSO).

The storage credentials stored in the app's database are only as safe as your NAS: anyone who
can read `/Volume1/DockerAppData/openlistdocker/` can read them. Keep that folder out of
shared folders and backups you do not control.

## 6. Third-party sharing and API disclosure

The developer shares nothing and receives nothing. The table lists every endpoint the app or
its web UI can contact, what is sent and where it goes. Rows marked “only when you use it”
are inactive by default until you configure or trigger that feature.

| Service | Endpoint | Data sent | Destination / region |
|---|---|---|---|
| A storage you mount | that provider's official API or SDK endpoint (for example `graph.microsoft.com`, `www.googleapis.com`, `api.aliyundrive.com`, `pan.baidu.com`, an S3 endpoint, or the WebDAV/SFTP/FTP/SMB address you enter) | the credentials/tokens you stored and the file operations you request (list, read, upload, move) | the provider you chose, or your own server |
| OAuth sign-in for such a drive, only when you add it | the provider's authorisation endpoint (for example `login.microsoftonline.com` or `accounts.google.com`) | the authorisation request; the token returned is stored in the app database | the provider's servers |
| Web UI static assets (always) | `cdnjs.cloudflare.com` (JavaScript polyfill), `res.oplist.org` and `cdn.oplist.org` (logo, default audio cover, PDF.js / EPUB.js viewers) | your browser's IP address and user agent, the asset requested — no account or file data | Cloudflare and the upstream project's CDN, outside your NAS |
| Office document preview, only when you open one | `view.officeapps.live.com/op/view.aspx` (Microsoft) or `docs.google.com/gview` (Google) | the preview link of the document you opened; that service fetches the document from it | Microsoft, US / Google, US |
| PDF / EPUB preview, only when you open one | `res.oplist.org/pdf.js` or `res.oplist.org/epub.js` viewers | the preview link of the file | the upstream project's CDN |
| OCR, only when you run it on an image or PDF | `openlistteam-ocr-api-server.hf.space/ocr/file/json` by default, or the OCR server address you configure | the image or PDF you submit for OCR | OpenListTeam's hosted OCR service (Hugging Face Spaces) — change it in Settings → OCR to use your own server |
| Single sign-on, only if you enable it | the provider you configure (GitHub, Google, Microsoft, DingTalk, Feishu, or your OIDC issuer) | the authentication request (client id, redirect URI, the provider account you sign in with) | that provider's servers |
| Book metadata, only if you add a Google Books API key | `www.googleapis.com/books` | the ISBN or title you search for | Google, US |
| WebDAV / API clients you point at the app | the address you give them | the file operations you perform | your LAN |
| Image pull at install | `registry-1.docker.io` or your registry mirror | anonymous pull | Docker Hub, US / your mirror |

No data is sold or shared with advertisers. There is no third-party analytics or tracking SDK
in the app. The upstream project states that it collects no personal data (see
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

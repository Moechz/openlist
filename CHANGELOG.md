# Changelog

All notable user-visible changes to the OpenList TOS application package.

## 4.2.9-1 — 2026-11-18

### Changed
- Upstream image `openlistteam/openlist:v4.2.9` (was `v4.2.2`). Since upstream
  `v4.2.7` the first start no longer prints a random administrator password to the
  container log: the web UI opens a setup wizard where the administrator account is
  created.
- The English description and `important` text now describe that setup wizard instead of
  a password printed in the log.
- The `zh-cn` and `zh-hk` descriptions were rewritten: the previous text described an
  unrelated application (habit tracking) and pointed at another application's data
  folder.
- `help` in `config.ini` now opens the privacy policy (`PRIVACY.md` in this repository).
- `PUID`/`PGID` were removed from the compose environment: the image never read them.
- The health check probes `/ping` instead of `/`, so it reports healthy before the setup
  wizard has been completed.

### Added
- `PRIVACY.md` and `NOTICE.md`, plus README sections for privacy, third-party services
  (API disclosure), licence and support — the remediation asked for by the listing review
  (items C3–C10).
- `docs/REVIEW-REPLY-4.2.9-1.md`, the item-by-item reply to that review.

### Removed
- The 6 MB `TOSAppSelfTestingTool` binary is no longer tracked in the public repository
  (it is a packaging tool, still used locally and now ignored by git).

## 4.2.2-2 — 2026-11-12

### Changed
- Icon: replaced the letter monogram with a functional vector mark that describes what the app does (the upstream project does not publish a compliant SVG).

## 4.2.2-1 — 2026-11-11

### Added
- First release. OpenList (`openlistteam/openlist:v4.2.2`, Docker Hub official image, unmodified)
  packaged as a TOS 7 Docker application, published on port 8086 and running
  as a non-root user (1000:1000).
- Persistent data is kept under `/Volume1/DockerAppData/openlistdocker/`.

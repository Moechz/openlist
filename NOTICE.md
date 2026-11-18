# NOTICE — third-party components and attributions

This package (`openlistdocker`, TOS 7 Docker application) is an independent packaging by
Moechz of the upstream **OpenList** project.

## 1. Upstream application — OpenList

- Project: <https://github.com/OpenListTeam/OpenList>
- Licence: **AGPL-3.0**
- Relationship: the upstream application is used **unmodified**, inside the container image.
  No upstream source code is redistributed in this package; the package only pulls the image
  from Docker Hub.
- Upstream privacy statement: <https://doc.oplist.org/privacy>
- Upstream terms of use: <https://doc.oplist.org/terms>

## 2. Container image — `openlistteam/openlist:v4.2.9`

- Registry: Docker Hub (`openlistteam/openlist:v4.2.9`)
- Maintainer: the OpenList Team
- The image is built by its maintainer from `openlistteam/openlist-base-image` (Alpine based)
  and is pulled from Docker Hub at install time; it is not redistributed here.

## 3. Components inside the image

The image bundles the Go application and its dependencies, plus the Alpine userland of its
base image. Their licences are documented in the image's own package metadata and in the
upstream repository; they are shipped inside the image only and are not redistributed as
separate files by this package.

## 4. This packaging

The files authored for this package — `docker-compose.yml`, `config.ini`,
`openlistdocker.lang`, `openlistdocker.svg`, the build/verification scripts and this
documentation — are released under the **MIT** licence (see `LICENSE`).

Running the AGPL-3.0 application from its official image does not make this packaging a
derivative work of it; the packaging contains no upstream code.

## 5. Trademarks

“OpenList”, “AList”, “TerraMaster” and “TOS” are trademarks or registered trademarks of their
respective owners. Their use here is descriptive only. This package is **not affiliated with,
sponsored or endorsed by** the OpenList Team or TerraMaster.

## 6. Icon

The application icon in `src/openlistdocker.svg` is an original mark drawn for this package
(a folder outline on a rounded blue tile). It does not reproduce the upstream logo.

## 7. Contact

Moechz — moechz@163.com — <https://github.com/Moechz/openlist>

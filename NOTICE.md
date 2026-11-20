# Notice

## Upstream software

This repository contains Railway configuration and a small bootstrap script only. Forgejo is pulled at build time from its official container image; it is not stored in this repository.

| Software | License | Source (publisher) |
|---|---|---|
| Forgejo v17.0.2 (with OpenSSH, s6 and the Alpine base the image ships) | GPL-3.0-or-later (Forgejo); the image's Alpine packages under their own licenses | `codeberg.org/forgejo/forgejo:17.0.2@sha256:879f4f10a98e3e167b32cde5f7ae3af0163d50bfacf567559a27a0e95a95670c` (Forgejo project) |
| PostgreSQL 18 | PostgreSQL License | `ghcr.io/railwayapp-templates/postgres-ssl:18.6@sha256:4fc833b43d2aaf6ecc9b18615f5533602b723a32d0e2efe2dae8ca0f7217c863` (Railway; MIT-licensed image repository) |

Forgejo's license: https://codeberg.org/forgejo/forgejo/src/tag/v17.0.2/LICENSE. Forgejo's source code for the pinned release: https://codeberg.org/forgejo/forgejo/src/tag/v17.0.2.

## Verified against

| What | Upstream file | Ref | Checked |
|---|---|---|---|
| Latest stable release | Codeberg releases (`v17.0.2`, 2026-11-18; `v15` is the LTS line) | `v17.0.2` | 2026-11-20 |
| Image layout, entrypoint, s6 services (gitea, openssh), first-boot `app.ini` template | `Dockerfile`, `docker/root/usr/bin/entrypoint`, `docker/root/etc/s6/*/run` and `setup`, `docker/root/etc/templates/app.ini`, `docker/root/usr/local/bin/gitea` | `v17.0.2` | 2026-11-20 |
| `FORGEJO__section__KEY` variables written to `app.ini` | `contrib/environment-to-ini/environment-to-ini.go`, `modules/setting/config_env.go` | `v17.0.2` | 2026-11-20 |
| `forgejo admin user create/list/change-password` flags | `cmd/admin_user_create.go` and related commands | `v17.0.2` | 2026-11-20 |
| Health endpoint `/api/healthz` | Forgejo API | `v17.0.2` | 2026-11-20 |
| Railway Postgres image (`18.6` = `18`) | ghcr.io tags of `railwayapp-templates/postgres-ssl` | 18.6 | 2026-11-20 |

## Adapted code

| Our file | Source | Changes |
|---|---|---|
| `services/forgejo/railway-admin` | Written for this template; follows the s6 service layout of Forgejo's image | — |

## Trademarks

"Forgejo" is a trademark of Codeberg e.V.; "Railway" and "PostgreSQL" belong to their respective owners. This is a community template and is not affiliated with or endorsed by them.

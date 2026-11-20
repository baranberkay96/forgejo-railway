# Forgejo for Railway: the official rootful image (Forgejo + OpenSSH under s6) plus one
# extra s6 service that creates the first administrator from Railway variables.
#
# Pinned by tag and digest. To upgrade, change both to a newer release:
# https://codeberg.org/forgejo/-/packages/container/forgejo/versions
FROM codeberg.org/forgejo/forgejo:17.0.2@sha256:879f4f10a98e3e167b32cde5f7ae3af0163d50bfacf567559a27a0e95a95670c

# s6-svscan (started by the image's CMD) supervises every directory in /etc/s6:
# gitea (the web server), openssh and our one-shot railway-admin.
COPY --chmod=755 railway-admin /etc/s6/railway-admin/run

# Fail the build early if the tools the bootstrap needs are missing.
RUN /usr/local/bin/gitea --version \
 && command -v su-exec curl s6-svc >/dev/null \
 && bash -n /etc/s6/railway-admin/run

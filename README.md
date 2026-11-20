# Deploy and Host Forgejo with Railway

Forgejo is a self-hosted software forge: Git hosting with pull requests, issues, wikis, code review, a package registry and CI integration, in one lightweight Go service. This community template runs Forgejo v17.0.2 with PostgreSQL 18, Git over HTTPS and SSH, Git LFS, and an administrator created for you on the first boot.

## About Hosting Forgejo

Forgejo needs one web service with persistent storage for repositories and a database. This template runs the official Forgejo container image (Forgejo plus OpenSSH), stores repositories, LFS objects, attachments and SSH host keys on a Railway volume, and keeps users, issues and pull requests in Postgres on the private network. The web installer is skipped: everything is configured from Railway variables, a secret key is generated at deploy, registration is closed, and the first administrator is created from variables once the server is healthy. SSH clone and push work through a Railway TCP proxy, and the clone URLs Forgejo shows already contain the proxy's host and port. Measured on Railway: about 0.25 GB of RAM for Forgejo and 0.13 GB for Postgres.

## Common Use Cases

- Private Git hosting for a team or a company, with pull requests and code review
- A GitHub or GitLab alternative you control, including mirrors of upstream repositories
- Issue tracking, wikis and project boards next to the code
- A package registry (npm, Maven, PyPI, Cargo, OCI containers and more) for internal artifacts
- Hosting open-source projects on your own domain

## Dependencies for Forgejo Hosting

- Forgejo v17.0.2 (`codeberg.org/forgejo/forgejo`, official image, pinned by digest)
- PostgreSQL 18 (Railway's `postgres-ssl` image)

### Deployment Dependencies

- Forgejo website and documentation: https://forgejo.org/docs/latest/
- Source code and releases: https://codeberg.org/forgejo/forgejo
- Container installation guide: https://forgejo.org/docs/latest/admin/installation/docker/
- Configuration cheat sheet: https://forgejo.org/docs/latest/admin/config-cheat-sheet/

### Implementation Details

| Service | Role | Public |
|---|---|---|
| Forgejo | Web UI, API, Git over HTTPS and SSH, LFS; volume at `/data` for repositories, LFS objects, attachments, avatars and SSH host keys | HTTPS domain and a TCP proxy for SSH |
| Postgres | Users, organizations, issues, pull requests and settings; private only, with a volume | no |

**First login**

1. Wait until the Forgejo service is healthy. The first deploy builds a thin image on top of the official one and creates the database tables, which takes a few minutes.
2. Open the Forgejo service → Variables and copy `FORGEJO_ADMIN_USERNAME` (default `forgejo-admin`) and `FORGEJO_ADMIN_PASSWORD` (generated).
3. Open the Forgejo service's public domain and sign in. The deploy log shows `[railway-admin] Administrator 'forgejo-admin' created`.
4. Change the administrator's e-mail address in Settings (the default is a placeholder), and add your SSH key under Settings → SSH / GPG keys.

`admin` is a reserved name in Forgejo, which is why the default is `forgejo-admin`. Invalid values never break the deploy: an unusable user name falls back to `forgejo-admin`, and a password shorter than 8 characters creates no administrator until you set a longer one and redeploy (registration stays closed, so nobody else can take the instance). The variables are used only while no administrator exists; after that, manage accounts in Forgejo. Forgot the password? Set a new `FORGEJO_ADMIN_PASSWORD` and `FORGEJO_ADMIN_PASSWORD_RESET=true`, redeploy, then set it back to `false`.

**Users and registration.** Sign-up is closed (`FORGEJO__service__DISABLE_REGISTRATION=true`). Create accounts in Site administration → User accounts, or set the variable to `false` to open registration. Public repositories are visible without login; set `FORGEJO__service__REQUIRE_SIGNIN_VIEW=true` for a fully private instance.

**SSH.** The template adds a Railway TCP proxy to port 22. Clone URLs look like `ssh://git@{proxy-host}:{proxy-port}/{owner}/{repo}.git` and are filled in automatically from `RAILWAY_TCP_PROXY_DOMAIN` and `RAILWAY_TCP_PROXY_PORT`. Don't need SSH? Delete the TCP proxy in the service's Networking settings; HTTPS clone and push keep working.

**Custom domain.** Add it in the Forgejo service's Settings → Networking, then set `FORGEJO__server__DOMAIN` and `FORGEJO__server__ROOT_URL` to it and redeploy.

**Configuration.** Every Forgejo setting can be set as a variable named `FORGEJO__{section}__{KEY}` (for example `FORGEJO__repository__DEFAULT_PRIVATE=private`); it is written into `app.ini` at each boot. Keep `FORGEJO__security__SECRET_KEY` unchanged: it encrypts two-factor secrets and other values in the database.

**E-mail.** Set `FORGEJO__mailer__ENABLED=true` and the `FORGEJO__mailer__*` settings for notifications and password resets. Railway allows outbound SMTP only on the Pro plan.

**Forgejo Actions.** Actions is available in the UI, but this template has no runner: Forgejo's runner executes jobs in Docker containers, and Railway services cannot run Docker. Register a runner on a machine or VM you control (Settings → Actions → Runners) to run workflows.

**Resources and plans.** Measured on Railway (Forgejo 17.0.2, 2026-11-20): Forgejo uses about 0.24 GB of RAM at idle and Postgres about 0.13 GB, so a small instance costs roughly $4-6 a month at Railway's usage prices. Every plan works, Free included; repositories grow the volume, so large or many repositories belong on Hobby or Pro.

**Scaling.** Forgejo runs as one replica because repositories live on the service's volume. For more users raise the service's CPU and memory; Postgres can be resized independently.

**Backups.** Repositories are on the Forgejo volume and metadata in Postgres; back up both (Railway volume backups, or `forgejo dump` from a shell).

**Upgrading Forgejo.** The version is pinned in `services/forgejo/Dockerfile` (tag and digest). When this template is updated, Railway offers the update. To pick a version yourself, eject the service to your own GitHub repository and change the `FROM` line to another release. Database migrations run automatically on start; read the release notes and back up before a major upgrade.

**License.** Forgejo is free software under GPL-3.0-or-later. This is a community template, not affiliated with the Forgejo project or Codeberg e.V.

### Why Deploy Forgejo on Railway?

Railway runs Forgejo and its Postgres database in one project with a private network, an HTTPS domain, a TCP proxy for SSH, persistent volumes, logs and usage-based pricing. This template gives you a locked-down, ready-to-use Git forge in a few minutes, with SSH clone URLs that work out of the box and no setup wizard to finish.

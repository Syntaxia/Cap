<p align="center">
  <p align="center">
   <img width="150" height="150" src="https://github.com/CapSoftware/Cap/blob/main/apps/desktop/src-tauri/icons/Square310x310Logo.png" alt="Logo">
  </p>
	<h1 align="center"><b>Cap (Syntaxia Fork)</b></h1>
	<p align="center">
		Syntaxia's self-hosted fork of <a href="https://github.com/CapSoftware/Cap">Cap</a>, the open source Loom alternative.
    <br />
    <a href="https://cap.syntaxia.com"><strong>cap.syntaxia.com »</strong></a>
  </p>
</p>
<br/>

This is Syntaxia's fork of [Cap](https://github.com/CapSoftware/Cap). We build and deploy our own Docker image from source so that bug fixes and customizations ship immediately, rather than depending on upstream's pre-built image.

## Deployment

Our instance runs at **https://cap.syntaxia.com** on a Digital Ocean droplet.

### How It Works

1. Push to `main` triggers GitHub Actions (`.github/workflows/deploy-cap.yml`)
2. CI builds the Docker image from `apps/web/Dockerfile`
3. Image is saved as a tarball, SCPed to the server, and loaded with `docker load`
4. `docker compose up -d --force-recreate` restarts services

No container registry is used — the image is transferred directly to the server.

### Secrets (1Password)

All secrets are stored in a 1Password vault and fetched at deploy time via the `op` CLI. The GitHub Actions workflow uses `OP_SERVICE_ACCOUNT_TOKEN` to authenticate. Secrets are written to `.env` on the server, consumed by `deploy/docker-compose.prod.yml`.

### Infrastructure

| Component | Details |
|-----------|---------|
| **Server** | Digital Ocean droplet, SSH user `deploy` |
| **Reverse Proxy** | Caddy (SSL via Let's Encrypt) |
| **Database** | MySQL 8.0 (Docker, volume `cap-mysql-data`) |
| **Media Server** | Upstream `ghcr.io/capsoftware/cap-media-server:latest` |
| **Storage** | AWS S3 (path-style URLs) |
| **Domain** | `cap.syntaxia.com`, signup restricted to `syntaxia.com` |

### Key Files

- `deploy/docker-compose.prod.yml` — Production compose (Caddy + app + media-server + MySQL)
- `deploy/Caddyfile` — Reverse proxy config
- `.github/workflows/deploy-cap.yml` — CI/CD pipeline
- `apps/web/Dockerfile` — Multi-stage build (accepts `NEXT_PUBLIC_WEB_URL` as build arg)

---

For upstream Cap documentation, see the [original README](https://github.com/CapSoftware/Cap#readme) and [self-hosting docs](https://cap.so/docs/self-hosting).

# Monorepo App Architecture

We use a combination of Rust, React (Next.js), TypeScript, Tauri, Drizzle (ORM), MySQL, TailwindCSS throughout this Turborepo powered monorepo.

> A note about database: The codebase is currently designed to work with MySQL only. MariaDB or other compatible databases might partially work but are not officially supported.

### Apps:

- `desktop`: A [Tauri](https://tauri.app) (Rust) app, using [SolidStart](https://start.solidjs.com) on the frontend.
- `web`: A [Next.js](https://nextjs.org) web app.

### Packages:

- `ui`: A [React](https://reactjs.org) Shared component library.
- `utils`: A [React](https://reactjs.org) Shared utility library.
- `tsconfig`: Shared `tsconfig` configurations used throughout the monorepo.
- `database`: A [React](https://reactjs.org) and [Drizzle ORM](https://orm.drizzle.team/) Shared database library.
- `config`: `eslint` configurations (includes `eslint-config-next`, `eslint-config-prettier` other configs used throughout the monorepo).

### License:
Portions of this software are licensed as follows:

- All code residing in the `cap-camera*` and `scap-*` families of crates is licensed under the MIT License (see [licenses/LICENSE-MIT](https://github.com/CapSoftware/Cap/blob/main/licenses/LICENSE-MIT)).
- All third party components are licensed under the original license provided by the owner of the applicable component
- All other content not mentioned above is available under the AGPLv3 license as defined in [LICENSE](https://github.com/CapSoftware/Cap/blob/main/LICENSE)
  
# Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for more information. This guide is a work in progress, and is updated regularly as the app matures.

## Analytics (Tinybird)

Cap uses [Tinybird](https://www.tinybird.co) to ingest viewer telemetry for dashboards. The Tinybird admin token (`TINYBIRD_ADMIN_TOKEN` or `TINYBIRD_TOKEN`) must be available in your environment. Once the token is present you can:

- Provision the required data sources and materialized views via `pnpm analytics:setup`. This command installs the Tinybird CLI (if needed), runs `tb login` when a `.tinyb` credential file is missing, copies that credential into `scripts/analytics/tinybird`, and finally executes `tb deploy --allow-destructive-operations --wait` from that directory. **It synchronizes the Tinybird workspace to the resources defined in `scripts/analytics/tinybird`, removing any other datasources/pipes in that workspace.**
- Validate that the schema and materialized views match what the app expects via `pnpm analytics:check`.

Both commands target the workspace pointed to by `TINYBIRD_HOST` (defaults to `https://api.tinybird.co`). Make sure you are comfortable with the destructive nature of the deploy step before running `analytics:setup`.

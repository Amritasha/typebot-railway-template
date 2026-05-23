# Typebot on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/template/typebot)

[Typebot](https://typebot.io) is an open-source chatbot builder that lets you create conversational forms and embed them anywhere — a self-hosted alternative to Typeform and Landbot.

## Architecture

Typebot requires two services plus a database:

| Service | Docker image | Purpose |
|---------|-------------|---------|
| **Builder** | `baptistearno/typebot-builder:3` | Admin UI to create and manage bots |
| **Viewer** | `baptistearno/typebot-viewer:3` | Runtime that renders bots to end users |
| **PostgreSQL** | Railway Postgres plugin | Shared database |

## Deploy

### Step 1 — Add PostgreSQL

In your Railway project, click **+ New → Database → PostgreSQL**. Copy the `DATABASE_URL` from its Variables tab.

### Step 2 — Deploy the Builder

1. Click **+ New → GitHub Repo** and point it at this repository.
2. Railway auto-detects `Dockerfile` and `railway.toml`.
3. Set these variables in the service's Variables tab:

| Variable | Value |
|----------|-------|
| `DATABASE_URL` | From the Postgres plugin |
| `NEXTAUTH_URL` | Your builder's public URL (set after first deploy) |
| `NEXT_PUBLIC_VIEWER_URL` | Your viewer's public URL (set after viewer deploy) |
| `ENCRYPTION_SECRET` | `openssl rand -hex 16` |
| `NEXTAUTH_SECRET` | `openssl rand -hex 32` |
| `ADMIN_EMAIL` | Your email (pre-authorized as admin) |

### Step 3 — Deploy the Viewer

1. Click **+ New → Docker Image** and enter `baptistearno/typebot-viewer:3`.
2. Set these variables:

| Variable | Value |
|----------|-------|
| `DATABASE_URL` | Same Postgres plugin URL |
| `ENCRYPTION_SECRET` | **Same value** as builder |
| `NEXT_PUBLIC_VIEWER_URL` | This viewer's public URL |
| `NEXTAUTH_URL` | Builder's public URL |
| `PORT` | `3000` |

### Step 4 — Wire the URLs

Once both services have public domains, go back to the builder and update `NEXTAUTH_URL` and `NEXT_PUBLIC_VIEWER_URL` with the correct URLs. Then redeploy both services.

## Environment variables

See [`.env.example`](.env.example) for the full list including SMTP, S3, and OAuth options.

### Generating secrets

```bash
# ENCRYPTION_SECRET (16 bytes hex = 32 chars)
openssl rand -hex 16

# NEXTAUTH_SECRET
openssl rand -hex 32
```

> Both services must share the **same** `ENCRYPTION_SECRET`. If they differ, bots will fail to render.

## Upgrading

The image version is pinned in [`Dockerfile`](Dockerfile) and [`Dockerfile.viewer`](Dockerfile.viewer). Change `3` to a specific tag (e.g. `3.5.0`) to pin a release.

```dockerfile
FROM baptistearno/typebot-builder:3.5.0
```

## License

Typebot is [AGPL-3.0](https://github.com/baptisteArno/typebot.io/blob/main/LICENSE). This template is MIT.

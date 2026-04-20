---
name: identify-deployment-arch
description: Analyze the current project to identify deployment architecture — where code runs, what services it depends on, where state lives. Use at the start of a backup-planning engagement, before deciding what to protect.
---

# Identify Deployment Architecture

Before a sensible backup strategy can be designed, you must understand what the system actually looks like in production. This skill produces a short architecture map that the later skills consume.

## Steps

1. **Inventory the runtime surface.** Read `README.md`, `docker-compose.yml`, `Dockerfile`, `*.tf`, `k8s/`, `fly.toml`, `vercel.json`, `render.yaml`, `Procfile`, `package.json` (scripts + deps), `pyproject.toml`, `requirements.txt`, `wrangler.toml`, `serverless.yml`, and any CI config. Note the deployment target(s): VPS, container host, serverless, PaaS, on-prem, hybrid.
2. **Identify persistent state.** For each service, classify storage:
   - Database (Postgres, MySQL, SQLite, Mongo, Redis-with-persistence, etc.) — note version and whether managed or self-hosted.
   - Object storage (S3, R2, B2, GCS, Azure Blob).
   - Block/file storage (volumes, NFS, bind mounts).
   - External SaaS (auth providers, CMS, analytics) that hold canonical data.
3. **Identify configuration and secrets.** `.env` files, secret managers, CI/CD secret stores, TLS certs, SSH keys bound to the deployment.
4. **Map data flow.** Briefly note: where user-facing writes land, what's derived vs canonical, what's ephemeral (caches, queues, build artefacts).
5. **Ask the user** to confirm anything ambiguous — especially whether the deployment is single-region / multi-region, and whether any storage is already replicated by the provider.

## Output

Write findings to `backup-plan/01-architecture.md` in the project. Use this structure:

```
# Deployment Architecture

## Runtime
- <service>: <where it runs>, <how it's deployed>

## Persistent State
- <store>: <type>, <location>, <provider-level durability>

## Configuration & Secrets
- <source>

## Data Flow Notes
- <canonical vs derived, ephemeral exclusions>
```

Keep it tight — one page. This document is input to `identify-data-to-protect`.

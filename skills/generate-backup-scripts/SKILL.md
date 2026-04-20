---
name: generate-backup-scripts
description: Generate the actual backup scripts and scheduling units (systemd timers, cron, GitHub Actions, Kubernetes CronJobs) that implement the documented strategy. Use after BACKUP-STRATEGY.md is approved.
---

# Generate Backup Scripts

Translate the strategy document into runnable code. Output lives in `backup-plan/scripts/` and, where relevant, in the project's existing `ops/`, `.github/workflows/`, or `k8s/` directories.

## Inputs

- `backup-plan/BACKUP-STRATEGY.md` — the source of truth. If it doesn't exist, stop and run `document-backup-strategy` first.

## Per-tool skeletons

Match the tool(s) the strategy picked. Always produce: (a) the backup script itself, (b) a matching restore script stub, (c) the scheduling unit, (d) a short README inside `backup-plan/scripts/` explaining how to run and verify.

### restic → S3-compatible

```bash
#!/usr/bin/env bash
set -euo pipefail

: "${RESTIC_REPOSITORY:?}"     # e.g. s3:https://s3.us-east-005.backblazeb2.com/<bucket>/<prefix>
: "${RESTIC_PASSWORD_FILE:?}"
: "${AWS_ACCESS_KEY_ID:?}"
: "${AWS_SECRET_ACCESS_KEY:?}"

TAG="${BACKUP_TAG:-$(hostname -s)}"
SOURCES=( /path/to/data /etc/<app> )
EXCLUDES=( --exclude-caches --exclude '*.tmp' --exclude 'node_modules' )

restic backup "${SOURCES[@]}" "${EXCLUDES[@]}" --tag "$TAG" --one-file-system
restic forget --tag "$TAG" --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --keep-yearly 3 --prune
restic check --read-data-subset=5%
```

### Postgres logical dump + restic

Dump to a FIFO and pipe into restic to avoid landing plaintext on disk:

```bash
restic backup --stdin --stdin-filename "${DB}-$(date +%F).sql" \
  --tag pg-${DB} \
  <(pg_dump --format=custom --no-owner --dbname "$DATABASE_URL")
```

### rclone sync (object → object mirror)

```bash
rclone sync --fast-list --checksum --transfers 16 \
  primary:bucket/prefix secondary:bucket/prefix \
  --backup-dir secondary:bucket/_versions/$(date +%F)
```

### systemd timer

Produce paired `<name>.service` (Type=oneshot) and `<name>.timer` (OnCalendar=...). Always include `Persistent=true` so missed runs fire on boot.

### GitHub Actions / Kubernetes CronJob

Generate idiomatic manifests. Inject secrets via repository secrets or `secretRef` — never bake credentials into the script.

## Hard requirements for every script

- `set -euo pipefail` (bash) or equivalent strict mode.
- Credentials come from env vars or a keyfile referenced from the environment. Never hard-coded.
- Exits non-zero on any failure so the scheduler records it.
- Logs to stdout/stderr; the scheduler captures. No custom logfiles in `/tmp`.
- A clear "dry run" path (`--dry-run` flag or `DRY_RUN=1` env).
- A corresponding restore-command comment at the top of the file.

## After generation

- Make scripts executable: `chmod +x`.
- Write a `backup-plan/scripts/README.md` listing each script, what it does, how to dry-run, and where logs land.
- Cross-link from `BACKUP-STRATEGY.md` → the scripts.
- Remind the user to test with `--dry-run`, then do a real run, then verify by listing snapshots / dumped objects.

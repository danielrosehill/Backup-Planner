---
name: save-backup-infra
description: Capture the user's available backup infrastructure (S3/B2/R2 buckets, rsync.net, NAS, external drives, existing restic repos, etc.) to persistent memory so it can be reused across projects without re-asking. Use when the user mentions backup destinations, or before evaluating backup options.
---

# Save Backup Infrastructure to Memory

The user likely has standing backup infrastructure — cloud object storage accounts, a home NAS, rsync.net quota, an existing restic/borg repo — that should be reused across every project rather than re-provisioned. Capture this once, then recall it for every future backup plan.

## What counts as infrastructure

- **Object storage accounts**: AWS S3, Cloudflare R2, Backblaze B2, Wasabi, iDrive e2, Storj
- **SSH-reachable targets**: rsync.net, Hetzner Storage Box, a personal VPS with spare disk, NAS over SSH
- **Local targets**: NAS (SMB/NFS), external USB drives, a backup workstation
- **Existing backup tooling**: restic/borg/kopia repos already in use, their locations and purpose
- **Offsite partners**: anyone else's infrastructure the user backs up to (or receives backups from)

## Migration from legacy path (one-time)

Before resolving the data directory, check for legacy data at these paths:
- `~/.claude/projects/-home-*/memory/reference_backup_*.md`

For each legacy file that exists AND where `<plugin-data-dir>/references/backup_<short-name>.md` does NOT exist, move it to the new location (renamed without the `reference_` prefix — e.g. `reference_backup_b2.md` → `backup_b2.md`) and delete the legacy file. Tell the user: "Migrated <n> backup-destination reference files from ~/.claude/projects/.../memory/ to <new>."

## Path resolution

Resolve the plugin's data directory as `$CLAUDE_USER_DATA/backup-planner/` if `CLAUDE_USER_DATA` is set; otherwise `$XDG_DATA_HOME/claude-plugins/backup-planner/` if `XDG_DATA_HOME` is set; otherwise `~/.local/share/claude-plugins/backup-planner/`. Create the directory (and a `references/` subdirectory) if it doesn't exist. See the canonical convention in the `meta-tools:plugin-data-storage` skill.

## Steps

1. **Ask the user** to list what they have available for this project. Phrase it as "what backup destinations do you already have provisioned" rather than "what do you want to use" — you want the superset, not a pre-filtered choice.
2. **For each, capture:**
   - Type and provider
   - Rough capacity / remaining quota
   - Region (for compliance/latency)
   - Access method (credentials location — never the credentials themselves)
   - Cost profile (egress-free? per-GB? flat?)
   - Whether it's already holding other backups (shared repo) or fresh
3. **Write one reference file per destination** under `<plugin-data-dir>/references/`. Filename pattern: `backup_<short-name>.md`.
4. **Deduplicate** — if a destination is already recorded, update the existing entry rather than creating a new one.

## Memory file template

```markdown
---
name: Backup destination — <short name>
description: <provider, type, what it's good for>
type: reference
---

- **Type:** <S3-compatible / SSH / NAS / etc.>
- **Provider / Host:** <e.g. Backblaze B2, rsync.net, 10.0.0.x>
- **Capacity / quota:** <e.g. 1 TB provisioned, ~400 GB free>
- **Region:** <e.g. eu-central, on-prem Jerusalem>
- **Access:** <where credentials live — 1Password item name, env var, ~/.config/...>
- **Cost profile:** <flat / per-GB / egress-free / etc.>
- **Current use:** <empty / holds restic repo "foo" / shared with project X>
- **Notes:** <quirks, e.g. "B2 application key is restricted to bucket XYZ">
```

## Recall

When starting `evaluate-backup-options` on a new project, read the `<plugin-data-dir>/references/backup_*.md` files first so recommendations draw from the real pool.

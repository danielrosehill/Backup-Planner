---
name: evaluate-backup-options
description: Recommend the best-fit backup strategy for the project given its data inventory and the user's available infrastructure. Produces a ranked options brief with trade-offs. Use after architecture and inventory are mapped.
---

# Evaluate Backup Options

Synthesize the data inventory and the available infrastructure into 2–3 concrete strategy options, with a clear recommendation.

## Inputs to load

- `backup-plan/01-architecture.md`
- `backup-plan/02-data-inventory.md`
- All `reference_backup_*` memory files (see `save-backup-infra`)

If any are missing, run the corresponding upstream skill first.

## Evaluation axes

For each candidate strategy, score against:

- **RPO/RTO fit** — does it meet the tiers in the inventory?
- **3-2-1 compliance** — 3 copies, 2 media, 1 offsite. Flag gaps.
- **Encryption** — at rest and in transit. Client-side encryption is preferred for untrusted destinations.
- **Immutability / ransomware resistance** — append-only repos, object-lock, versioning.
- **Cost** — storage + egress + per-request. Factor restore cost, not just storage.
- **Operational complexity** — who runs it, how it's monitored, how restore is tested.
- **Vendor lock-in** — is the repo format portable (restic, borg) or proprietary?

## Tooling shortlist (match to data type)

- **Filesystem / mixed files:** restic, borg, kopia, rclone (sync-only, no snapshots)
- **Postgres:** `pg_dump` / `pg_dumpall` for logical, `pg_basebackup` + WAL for PITR, or managed provider snapshots
- **MySQL/MariaDB:** `mysqldump`, `mariabackup`, or managed snapshots
- **SQLite:** `VACUUM INTO` or file copy with WAL checkpoint
- **MongoDB:** `mongodump`, or Ops Manager
- **Redis (persistent):** RDB + AOF copy
- **Object storage:** provider-native versioning + cross-region replication, or rclone to a second provider
- **Kubernetes:** Velero for cluster + PV backup
- **VMs:** provider snapshots, or Proxmox Backup Server / Veeam
- **Config / secrets:** encrypted repo (restic/borg) or dedicated secret-manager export

## Output

Write to `backup-plan/03-options.md`:

```
# Backup Strategy Options

## Option A — <name>
- Tooling: ...
- Destinations: ...
- RPO/RTO delivered: ...
- 3-2-1 status: ...
- Monthly cost estimate: ...
- Pros / Cons

## Option B — ...

## Recommendation
<which option, why, what open questions remain>
```

End with an explicit ask to the user to approve one option before moving to `document-backup-strategy`.

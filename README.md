# Backup-Planner

A Claude Code plugin that walks the user end-to-end through planning, documenting, and implementing a backup and data-protection strategy for the current project.

It assumes nothing about the stack — it reads the repo, asks what matters, remembers what infrastructure the user already has available (so it doesn't re-ask on the next project), recommends a fit-for-purpose strategy, and generates the scripts and schedules that implement it.

## Skills

- `identify-deployment-arch` — map the runtime and persistent state of the project.
- `identify-data-to-protect` — tier each data asset with RPO and RTO targets.
- `save-backup-infra` — persist the user's available backup destinations (S3/B2/R2, rsync.net, NAS, existing repos) to memory for reuse across projects.
- `evaluate-backup-options` — produce a ranked brief of 2–3 strategy options with trade-offs.
- `document-backup-strategy` — write the authoritative `BACKUP-STRATEGY.md`.
- `generate-backup-scripts` — materialise the scripts, schedules, and CI jobs that implement the strategy.
- `restore-plan` — the runbook and drill schedule for the half of backups that usually gets skipped.

## Typical flow

```
identify-deployment-arch
  → identify-data-to-protect
    → save-backup-infra            (may run earlier; results cached in memory)
      → evaluate-backup-options
        → document-backup-strategy
          → generate-backup-scripts
            → restore-plan
```

All outputs land under `backup-plan/` in the current project.

## Installation

```bash
claude plugins marketplace add danielrosehill/Claude-Code-Plugins
claude plugins install backup-planner@danielrosehill
```

## License

MIT

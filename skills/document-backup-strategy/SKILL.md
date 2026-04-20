---
name: document-backup-strategy
description: Write the authoritative backup strategy document for the project — the approved option from evaluation, spelled out with schedules, retention, responsibilities, and monitoring. Use after the user has approved an option.
---

# Document Backup Strategy

Produce the living document that every future "how do we back this up" question refers back to. Written once the user has picked an option from `03-options.md`.

## Required sections

Write to `backup-plan/BACKUP-STRATEGY.md` (capitalised so it's visible). Include:

1. **Summary** — one paragraph: what's protected, where it goes, who owns it.
2. **Scope** — what is in scope, what is explicitly out of scope (and why).
3. **Data classes and targets** — table mapping each asset → RPO → RTO → tool → destination → retention.
4. **Schedule** — when each job runs (cron expression or equivalent), approximate runtime, when it must not run (backup windows).
5. **Retention policy** — grandfather-father-son or forget-policy in tool-native terms (e.g. restic `--keep-daily 7 --keep-weekly 4 --keep-monthly 12 --keep-yearly 5`). Align with compliance requirements.
6. **Encryption & key management** — where keys live, who can decrypt, what happens if the primary operator is unavailable.
7. **Monitoring & alerting** — how success/failure is detected, where alerts go, the SLO for responding to a failed job.
8. **Access and credentials** — which accounts/keys each job uses, least-privilege notes (e.g. restricted B2 application key, IAM policy scoped to one bucket/prefix).
9. **Testing cadence** — when restores are drilled (see `restore-plan` skill). Untested backups are not backups.
10. **Change log** — initial entry with today's date and approving user.

## Quality bar

- Every schedule is concrete (no "regularly" / "periodically").
- Every destination references a real entry from the backup-infra memory.
- Every asset from `02-data-inventory.md` is either covered or listed as out-of-scope with justification.
- No secrets in the document. Reference credential locations by name only.

After writing, append a pointer to the main project `README.md` under a "Backups" heading so future contributors find it.

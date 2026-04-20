---
name: restore-plan
description: Document and rehearse the restore procedure — the other half of a backup that most plans skip. Produces a runbook and a testing schedule. Use after scripts are generated, and whenever a drill is due.
---

# Restore Plan & Drill

A backup that has never been restored is a rumour. This skill produces the restore runbook and the drill log.

## Runbook — write to `backup-plan/RESTORE-RUNBOOK.md`

Structure:

1. **Scenario catalogue** — cover at minimum:
   - Single-file restore (accidental delete)
   - Single-table / single-database restore
   - Full-host loss (hardware gone)
   - Region / provider loss (primary destination unavailable)
   - Ransomware / malicious deletion (need an immutable copy)
2. **Per-scenario steps** — for each, a numbered procedure that assumes the operator has nothing except the runbook, the credentials vault, and the destination:
   - Prerequisites to obtain (credentials, DNS, new infrastructure)
   - Exact commands to list available snapshots / dumps
   - Exact commands to restore to a scratch location
   - Verification step (checksum, row count, application smoke test)
   - Cutover step (if restoring into production)
3. **Decision tree for partial vs full restore** — when to point-in-time vs latest.
4. **Expected duration** — measured, not guessed (from the last drill).
5. **Communication template** — who to tell, what to say, in a real incident.

## Drill schedule

Add a drill cadence to `BACKUP-STRATEGY.md`:

- **Critical tier:** full restore drill quarterly.
- **Important tier:** spot-check restore (random file / row) monthly; full drill annually.
- **Nice-to-have:** annual spot-check.

## Drill execution

When running a drill:

1. Open `backup-plan/drills/<YYYY-MM-DD>-<scenario>.md`.
2. Follow the runbook verbatim — if a step is wrong or missing, that is the finding. Do not improvise; fix the runbook instead.
3. Record: start time, end time, snapshot used, commands run, verification result, deviations from runbook, issues found.
4. Update `RESTORE-RUNBOOK.md` with corrections discovered.
5. Update the strategy doc's change log with the drill date and outcome.

## Integrity checks that aren't restore drills

These don't replace drills but should run on schedule:

- `restic check` / `borg check` on every repo — weekly.
- Test decryption of the latest snapshot against a known-good checksum — weekly.
- Verify alerting works: deliberately fail a job in a staging copy and confirm the alert fires — quarterly.

A backup that passes these and has a successful recent drill is a backup. Anything else is hope.

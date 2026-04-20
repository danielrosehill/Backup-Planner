---
name: identify-data-to-protect
description: Inventory the data assets that need backup protection — databases, volumes, user uploads, configs, secrets — and assign each a criticality tier, RPO, and RTO. Use after the deployment architecture is mapped.
---

# Identify Data to Protect

Turn the architecture map into a concrete protection inventory. Every item gets a tier, a target RPO (how much data loss is acceptable), and a target RTO (how fast recovery must complete).

## Steps

1. **Read `backup-plan/01-architecture.md`** if it exists. If not, run `identify-deployment-arch` first.
2. **List every candidate asset** — databases, volumes, object buckets, config files, secrets, generated media, logs that have audit value.
3. **For each, record:**
   - **What it is** (path, connection string placeholder, bucket name)
   - **Size & growth rate** (ballpark — ask the user if unknown)
   - **Canonical vs derivable** (can it be regenerated from other sources? If yes, consider excluding.)
   - **Tier**: `critical` (business-ending loss), `important` (painful loss), `nice-to-have` (re-creatable with effort)
   - **RPO target** (e.g. 1 hour, 24 hours, 7 days)
   - **RTO target** (e.g. 1 hour, 4 hours, next business day)
   - **Compliance/retention constraints** (GDPR, tax records, contractual)
4. **Explicit exclusions** — caches, node_modules, build artefacts, logs without audit value, anything reproducible from source.
5. **Flag anything the user hasn't considered** — secrets that would block restore, DNS records, account-level configuration in SaaS.

## Output

Write to `backup-plan/02-data-inventory.md`:

```
# Data Protection Inventory

## Critical
- <asset> — <size>, RPO <x>, RTO <y>, <notes>

## Important
- ...

## Nice-to-have
- ...

## Excluded (and why)
- ...

## Restore Prerequisites
- <secrets, DNS, account config that must exist for a restore to succeed>
```

This document drives `evaluate-backup-options`.

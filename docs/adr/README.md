# Architecture Decision Records

An ADR records one decision: why we made it, and what it costs.

## How we use them

- One decision per file. The file name is `NNN-short-title.md`.
- New records start from [`000-template.md`](000-template.md).
- **An accepted ADR is not edited.** If the decision changes, we write a new ADR. The new
  one says `Supersedes NNN`, and the old one gets `Superseded by NNN`.
- A question starts as an issue with the `adr-candidate` label. The ADR answers it. Then
  the issue is closed with a link to the ADR.

## Index

| # | Decision | Status | Date | Superseded by |
|---|---|---|---|---|
| [001](001-database-backup.md) | Do not build a backup system for SQLite in MVP | Accepted | 2026-08-14 | — |
| [002](002-ssh-authentication.md) | SSH authentication for Ansible | Accepted | 2026-08-23 | — |
| [003](003-bootstrap-model.md) | Bootstrap model: from a rented VPS to a managed node | Accepted | 2026-08-23 | — |
| [004](004-xray-config-source-of-truth.md) | The source of truth lives in `config.json`, not in SQLite | Accepted | 2026-09-11 | — |

This table is the only place that shows which decisions are in force. Update it in the
same commit that adds or supersedes an ADR.

## Open questions

These need a decision. Each one is a candidate for the next record.

- Hard delete or a disabled flag for a revoked credential.
- Service accounts and privilege separation on the managed node.
- Host-native, no Docker.
- The Telegram bot as the only control interface.

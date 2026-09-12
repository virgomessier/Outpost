# 004. The source of truth lives in `config.json`, not in SQLite

## Status
Accepted | 2026-09-11

## Context
Outpost has one node, one administrator and one inbound. The Definition of Done
requires these operations: issue, revoke, shortId rotation and a list of clients. An
early draft of `architecture.md` said that the desired state lives in SQLite, and that
something local applies the changes to Xray. We wrote that draft before we understood
how Xray receives changes.

Xray runs from `config.json`. It does not start without this file. `config.json` holds all
Xray settings: the inbound, the port, the REALITY private key, the shortIds and the
list of clients. The file that holds this state is on the node. It is there whether or
not another storage exists.

Xray has no reload. A restart re-reads `config.json`. The gRPC API changes state only in
memory, and it has no way to write that state back to `config.json`. `AlterInbound`
supports two operations only: add a user and remove a user. It cannot change shortIds.
`rmu` removes a user from the validator, but it does not close connections that are
already established.

Outpost has to remember a small set of facts: the REALITY private key, the shortIds,
the inbound parameters and one record per user. A user record is a UUID and a label. In
`config.json` the label is the `email` field of a client. This field is free text. It is
also the name that the gRPC API uses to address a user. The REALITY public key is not
stored anywhere. Xray computes it from the private key. ADR-001 decided that Outpost
has no backup. This state lives in one copy, on the server.

## Decision
The Outpost state lives in `config.json`. There is no other storage.

This decision removes other tools, such as a database. `config.json` is already Xray's
own file. It holds all the state that Outpost needs.

**Rejected options:**

1. The state lives in a database — the same facts then exist in two places, and nothing
   detects when the two disagree.

2. The state lives in `state.json` — `config.json` already holds every fact that Outpost
   has to remember, and a second JSON file holds the same facts in the same form, so it
   adds a copy and no new ability.

## Consequences

1. What became easier:

- The state lives in one place. Every question about the current users has one answer,
  and it comes from the same file that Xray reads.
- Outpost has no database. There is no schema, there are no migrations, and there is no
  second service to install and to keep alive.
- The state survives a restart and a reboot on its own. Xray reads the file at start, so
  nothing has to replay the users into it.
- The REALITY public key needs no storage. Xray computes it from the private key.

2. What became worse:

- `config.json` is a config file and a store at the same time. Every change rewrites the
  whole file. A bug in the writer can destroy the inbound and the REALITY private key,
  not only one user record.
- There are no transactions. A database gives them for free. The manager has to make sure
  that only one writer touches the file, or two commands overwrite each other.
- Secrets and state live together. The REALITY private key sits in the file that the
  manager rewrites many times. The owner and the permissions of that file matter more
  than before.
- The format belongs to Xray, not to Outpost. If Xray changes the shape of `config.json`,
  the storage of Outpost changes with it.
- The file keeps the current state only. There is no record of who was added and when.

3. What we must do now:

- Rewrite `architecture.md`. SQLite is not part of Outpost.
- Read the glossary again. `config.json` is the desired state now, so **Desired state**
  and **Reconciliation** need new definitions or have to go.
- Make outpost-manager the only writer of `config.json`, and let it write from one place
  at a time.
- Set the owner and the permissions of `config.json`. It holds the REALITY private key.
- Give every credential a stable and unique `email` label. That field is the only handle
  for a user.
- Write the ADR for the safe write protocol, and the ADR for the first start of a node.

Related: [ADR-001](001-database-backup.md)

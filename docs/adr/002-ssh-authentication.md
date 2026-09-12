# 002. SSH authentication for Ansible

## Status
Accepted | 2026-08-23

## Context
Ansible runs on a GitHub Actions runner, not on the administrator's machine. The
concept promised that the administrator installs no tools locally, so the runner had
to reach a freshly rented VPS on its own.

The runner needed a credential that survives between workflow runs, so the credential
had to be stored somewhere permanent. GitHub Secrets was the only storage available to
the workflow.

VPS providers hand over access in different ways. Some accept an SSH public key in the
order form. Cheaper providers only send a root login and a password by email. The
target audience is not a Linux administrator, so every extra setup step was expensive.

## Decision
The administrator owns one SSH key pair, dedicated to Outpost. The private key is stored
in GitHub Secrets and on the administrator's own machine. The public key is placed on
the VPS before the first workflow run, by any means the provider offers. Ansible
authenticates with that key and never with a password.

Placing the public key is one step for the administrator, and it has two recipes: the
"SSH keys" field in the provider order form, or `ssh-copy-id` from their machine. Both
end in the same state, so the playbook does not need to know which one was used.

**Rejected options:**

1. Ansible connects with the root password read from GitHub Secrets — this stores a
   reusable server password in a third party system and keeps password authentication
   enabled, while saving the administrator no steps: they already open a terminal to
   generate the key pair.

2. Ansible logs in with the root password, generates the key pair on the server, and
   the administrator copies it into GitHub Secrets — this needs the root password
   before it can start, so it adds a secret instead of removing one.

3. The runner generates the key pair and prints the private key in the job summary, so
   that the administrator can paste it into GitHub Secrets — job logs can be read and
   are kept, while secrets cannot be read back. A key that reaches a log is
   compromised.

4. Reuse the default user of the provider cloud image (`ubuntu`) — that user is created
   by cloud-init in specific provider images, not by Ubuntu itself, and cheap VPS
   images do not have it.

**Deferred options:**

- The runner generates the key pair and stores the private key with `gh secret set`,
  using a fine-grained PAT that the administrator creates once and deletes after use.
  This removes the terminal from the setup, which is what the audience needs. It also
  costs a second workflow, and it takes manual SSH access away from the administrator
  unless an optional `ADMIN_PUBLIC_KEY` secret is added. Waiting for a working MVP:
  until the deployment runs end to end, a second setup path is not worth the work. See
  issue #1.

## Consequences

1. What became easier:

- One credential to manage. Rotating it means replacing one GitHub secret and one
  `authorized_keys` entry.
- Password authentication is never needed, so the server can refuse passwords from the
  first run instead of from the end of the first run.
- The playbook has no branch for "password or key", which keeps it idempotent.
- The administrator keeps their own copy of the key, so a broken workflow does not lock
  them out of their server.

2. What became worse:

- The private key exists in two places: GitHub Secrets and the administrator's machine.
  A stealer on that machine is now a path to the server.
- The setup requires a terminal and one `ssh-keygen` command. This raises the skill bar
  above what `concept.md` promises for the audience. See issue #1.
- Whoever controls the repository controls the server. This is inherent to deploying
  from CI and cannot be mitigated inside the playbook.

3. What we must do now:

- Document both key placement recipes in `deployment.md`.
- Add `SSH deployment key (private)` to the asset table in `threat-model.md`, with both
  storage locations.
- Add the accepted risk "whoever controls the repository controls the server" to
  `threat-model.md`.
- Set `no_log: true` on every Ansible task that handles a secret.
- Pin third party GitHub Actions by commit SHA.

Related: [ADR-003](003-bootstrap-model.md)

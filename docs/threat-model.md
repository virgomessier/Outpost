# Threat model

This document lists what Outpost protects, what it protects against, and which risks it
accepts. Outpost has one managed node and one administrator, so the model is small.

## Assets

| Asset | Where it lives |
|---|---|
| SSH deployment key (private) | GitHub Secrets, and the administrator's own machine |
| REALITY private key | `config.json` on the managed node |
| Credentials (the user UUIDs) | `config.json` on the managed node |
| Telegram bot token | GitHub Secrets, and on the managed node |
| Administrator Telegram user ID | not a secret, but every bot command depends on it |

The REALITY public key is not an asset. Xray computes it from the private key.

## Threats

### Guessing SSH credentials

- Ansible authenticates with a key. `PasswordAuthentication no` is set, so the server
  refuses passwords from every account.
- `PermitRootLogin prohibit-password` is set. Root can log in with a key, but not with a
  password. [ADR-003](adr/003-bootstrap-model.md) explains why root stays open.
- `deploy-user` has no password, and the account is locked. The key is the only way in.
- Only the ports that Outpost needs are open.
- Fail2ban watches the sshd log and bans the addresses that fail to log in.

### Unauthorized control of the bot

- The bot answers in a private chat only.
- The bot checks the administrator by an immutable Telegram user ID.
- The username is never used for authorization. A username can change owner.
- The bot token is not stored in the repository.

### Scanning public ports

- Only the SSH port and the Xray port are open.
- A failed REALITY handshake does not give access to the proxy. The client sees the
  cover website.
- A credential never appears in a log.

## Accepted risks

These risks stay. We know about them, and we do not fix them in the MVP.

- **Whoever controls the repository controls the server.** This is inherent to deploying
  from CI. Nothing in the playbook can prevent it. See
  [ADR-002](adr/002-ssh-authentication.md).
- **The private SSH key exists in two places**: GitHub Secrets and the administrator's
  machine. A stealer on that machine is a path to the server.
- **Root can log in over SSH with a key.** A security checklist marks this as a problem.
  The same key already reaches root through `sudo`, so closing the direct way takes no
  path away from an attacker.
- **There is no backup.** If the server is gone, the REALITY private key and every
  credential are gone with it. Recovery means a new server and new links. See
  [ADR-001](adr/001-database-backup.md).
- **Outpost does not stop a distributed DDoS.** The project can only reduce the damage
  with local limits and with the tools of the VPS provider.
- **The server IP can get banned.** The fix is a new server, not a new configuration.

## Not covered yet

These need a decision before this document can cover them.

- Who runs each service, and with which privileges. This is an open question in the
  [ADR index](adr/README.md).
- The owner and the permissions of `config.json`. The file holds the REALITY private
  key, and outpost-manager rewrites it on every change.
- How outpost-manager writes `config.json` safely, and what happens when the restart
  fails. See issue #4.
- Who creates the first `config.json`, and when. See issue #7.
- What happens to a revoked credential. See issue #9.
- The bot must reach the Telegram API directly. If the traffic of the node ever goes
  through Xray, the bot cannot report a failure of Xray.
- Outpost does not run the Xray gRPC API. If it is ever enabled, that port has no
  authentication, and anyone who reaches it controls Xray.

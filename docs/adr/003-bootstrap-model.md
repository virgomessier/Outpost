# 003. Bootstrap model: from a rented VPS to a managed node

## Status
Accepted | 2026-08-23

## Context
ADR-002 decided how Ansible logs in. It did not decide what the first run does on the
server.

A new VPS had only one account: `root`. Only `root` could create other accounts. But
services must not run as `root`. So the playbook had to create its own user first.

The workflow must be safe to run again. `scope.md` says that a second run changes
nothing on a healthy server. This made the step from `root` to the new user a problem.
If the playbook closes the door it came through, every next run must check the state
of the server first.

Outpost has one administrator. The new user needs `sudo` without a password, because
nobody can type a password during a workflow run. The same private key opens both
`root` and the new user. There was no second key and no second person.

## Decision
The playbook has two plays.

- The first play connects as `root`. It does the bootstrap only: it creates
  `deploy-user`, installs the administrator's public key, adds `sudo`, and sets up
  `sshd`.
- The other plays connect as `deploy-user` and use `become` for privileged tasks.

The `deploy-user` account has no password. The account is locked, so the only way in
is the key. `sudo` is granted by a drop-in file in `/etc/sudoers.d/` with
`NOPASSWD:ALL`, because nobody can type a password during a workflow run.

`sshd` gets two settings: `PasswordAuthentication no` and
`PermitRootLogin prohibit-password`. Root stays open, but only for a key.

We keep root open on purpose. The same private key already reaches root through
`deploy-user` and `NOPASSWD:ALL`. Closing the direct way does not take any way away
from an attacker. It only stops the first play from running again. We pay a small loss
of strictness and get a playbook that is the same on every run.

**Rejected options:**

1. Run the whole playbook as `root` — a mistake in a task then runs with full
   privileges and nothing stops it, while Ansible needs no privilege that `sudo`
   cannot give.

2. Set `PermitRootLogin no` after the bootstrap — the playbook cannot run the first
   play a second time, so it needs a state check or a second workflow, and an attacker
   with the key still reaches root through `sudo`.

3. Give `deploy-user` a random password and store it nowhere — a password that nobody
   knows cannot help recovery, so it only adds one more thing that can leak.

4. Give `deploy-user` a random password and store it in GitHub Secrets — this puts a
   password back into the deployment path, which ADR-002 removed on purpose.

## Consequences

1. What became easier:

- The playbook is the same on every run. The first play always works, so there is no
  check for the state of the server.
- Root is used in one place only. Everything after the first play runs as
  `deploy-user`.
- The administrator has two ways back in: SSH with the key, and the provider console.
  `sshd` settings do not affect the provider console.

2. What became worse:

- Root can still log in over SSH with a key. A security checklist marks this as a
  problem, and we have to explain the reason every time.
- A mistake in the `sshd` config can lock out the runner and the administrator at the
  same time. Both of them come in over SSH.
- Two plays in one file are easy to misread. A task added to the first play runs as
  `root` by mistake.

3. What we must do now:

- Ansible must check the `sshd` config before it installs the file. Use
  `validate: /usr/sbin/sshd -t -f %s`.
- Write the `sshd` settings in a drop-in file in `/etc/ssh/sshd_config.d/`. Do not edit
  `/etc/ssh/sshd_config`. Ubuntu reads that directory first, and the first value of a
  key wins.
- Remember that Ubuntu 24.04 starts `sshd` from `ssh.socket`. `Port` and
  `ListenAddress` come from the socket unit, not from `sshd_config`. If Outpost ever
  changes the SSH port, it must edit a drop-in for `ssh.socket`.
- Add a comment at the top of the first play: this is the only play that runs as
  `root`.

Related: [ADR-002](002-ssh-authentication.md)

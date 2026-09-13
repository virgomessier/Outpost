# Architecture

Outpost runs on one managed node. Every component lives on that node.

## Components

**outpost-manager** — holds the business logic for users and credentials. It is the only
component that changes the state. It writes `config.json` and restarts Xray.

**Telegram bot** — checks the Telegram user ID of the administrator and passes commands
to outpost-manager. It does not touch Xray or `config.json` directly.

**Xray** — serves client connections. It is the only component that handles user
traffic. It reads `config.json` when it starts.

## State

`config.json` holds the whole state: the inbound, the port, the REALITY private key, the
shortIds and the list of clients. Outpost has no database. See
[ADR-004](adr/004-xray-config-source-of-truth.md).

outpost-manager is the only writer of this file.

## How a change is applied

1. The administrator sends a command to the bot.
2. The bot checks the Telegram user ID and passes the command to outpost-manager.
3. outpost-manager writes a new `config.json`.
4. outpost-manager restarts Xray.

Xray has no reload, so every change needs a restart. A restart closes the open
connections of every user. New connections work at once.

## Not decided yet

- How outpost-manager replaces the file, and what happens when the restart fails. See
  issue #4.
- Who creates the first `config.json`, and when. See issue #7.

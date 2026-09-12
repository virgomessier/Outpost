# Scope

## In scope (MVP)

- One managed node (1 vCPU, 1 GB RAM)
- Host-native (no Docker)
- One administrator
- One protocol profile (VLESS + REALITY + XTLS Vision)
- GitHub Secrets + Ansible run from a GitHub Actions runner
- Telegram bot as the control panel

## Out of scope

- Multi-node
- User accounts (logins and passwords)
- Web control panel

## Not now

- XHTTP + TLS/IP certificates
- AmneziaWG/WireGuard
- Automatic update of connection links in client apps after a revoke or a new issue
- Backup and restore of SQLite
- Nginx as a reverse proxy

## Definition of done

One run of the Ansible workflow in GitHub Actions deploys the managed node and the
Telegram bot that the administrator uses to control Outpost.

Besides deploying the architecture, the workflow must perform a basic security setup
on the managed node:

- Create a new `deploy-user` account in the sudo group
- Set `PermitRootLogin prohibit-password` and disable password authentication: SSH key authentication only
- Close every port that is not required; open only the ports needed for SSH and Xray
- Set correct permissions on directories and executable files
- Fail2ban watches the sshd logs and bans (with nftables rules) the addresses that try to log in and fail

After the deployment, the administrator can do the following in the Telegram bot:

- Issue a credential and a connection link
- List the issued credentials
- Revoke a credential
- Rotate shortIds
- Reboot the managed node; all services start again automatically

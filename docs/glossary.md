# Glossary

Terms used across Outpost documentation. One term means one thing everywhere: in the
documents, in the code, in commit messages and in the bot interface.

## People and access

**Administrator** — the person who controls Outpost through the Telegram bot. Outpost
has exactly one administrator. The administrator is identified by an immutable
Telegram user ID, never by a username.

**User** — a person who is allowed to connect to the proxy. A user has no account, no
password and no interface. A user only receives a connection link.

**Client app** — the software on the user's device that opens a connection link
(v2rayNG, Hiddify, Nekoray and others). Outpost does not ship a client app.

## Access material

**Credential** — the secret that identifies one user to Xray. In the MVP profile this
is a UUID. A credential belongs to exactly one user and never appears in logs.

**Connection link** — the `vless://` URI that the bot sends to the administrator. It
combines a credential with public server parameters (address, port, SNI, REALITY
public key, shortId). The Xray community also calls it a share link or a VLESS URI.

**Issue** (a link) — create a user, generate a credential, and return a connection
link.

**Revoke** — stop a credential from working. Outpost changes `config.json` and restarts
Xray, so the connection link no longer opens a session. New connections fail at once; a
connection that is already open continues until it closes on its own. Revoking does not
delete the old link from the user's device or from the chat history.

## Proxy stack

**Proxy** — Outpost forwards selected application traffic through the server. It does
not create a system-wide network tunnel.

**Xray** — the proxy server software (Xray-core). It is the only component that
handles user traffic.

**Inbound** — an Xray endpoint that listens for connections. An inbound defines a
port, a protocol, a transport and the list of clients allowed to use it. The MVP has
one inbound.

**Protocol profile** — the fixed combination that Outpost deploys: VLESS over RAW,
with REALITY as the security layer and XTLS Vision as the flow. The MVP ships one
profile and offers no choice.

**VLESS** — the proxy protocol used by Xray. VLESS has no encryption of its own; it
relies on the security layer below it.

**REALITY** — a TLS camouflage layer. The server borrows the TLS handshake of a real
third-party website, so an observer sees a connection to that website. REALITY needs a
key pair and a list of shortIds.

**shortId** — a short hex string configured on the inbound. A client must present one
of the configured shortIds to complete the REALITY handshake. It is a server
parameter, not a per-user secret.

**XTLS Vision** — the flow (`xtls-rprx-vision`) that reduces the "TLS inside TLS"
traffic pattern that DPI can detect.

**RAW** — the plain TCP transport in Xray. It was named `tcp` before Xray-core
v24.9.30; both names still work in configuration files.

**DPI** — deep packet inspection: network equipment that inspects traffic content and
can block it by protocol signature.

**TSPU** — the DPI system operated by the Russian state. TSPU is one specific
implementation of DPI.

## Deployment

**Deployment** — one run of the GitHub Actions workflow that brings a VPS from its
initial state to a working Outpost server.

**`deploy-user`** — the user that Ansible creates on the managed node in the first
play. Every later play connects as `deploy-user`. It has no password and gets `sudo`
through a `NOPASSWD` drop-in file.

**Template repository** — the public Outpost repository that a user copies to get a
private repository with their own secrets.

**Control node** — the machine that runs Ansible. In Outpost the control node is the
GitHub Actions runner, not the user's computer.

**Managed node** — the VPS that Ansible configures over SSH. Outpost manages one node.

**Idempotent** — running the deployment twice produces the same result as running it
once, and does not disturb a server that is already working.

## Words Outpost does not use

**VPN** — people search for this word, and Outpost solves the same problem for them,
but Outpost is technically a proxy. The documentation says *proxy*.

**Subscription** — in other panels this means a URL that returns an updating list of
configurations. Outpost does not have this feature. A single `vless://` URI is a
*connection link*.

**Account** — users have no login and no password, so they have no account. The word
is *user*.

**Client** — ambiguous: the Xray configuration calls user records `clients`, while
people call their app a client. Use *user* for the person and *client app* for the
software. Use *client* only when quoting an Xray configuration field.

**Key** — too vague. There are three different keys in Outpost: the SSH deployment
key, the REALITY private key and the REALITY public key. Always name which one.

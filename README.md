# aiSecurityWarden for Hermes Agent

[![Hermes compatibility](https://github.com/SoftDevEng/aiSecurityWarden-client/actions/workflows/hermes.yml/badge.svg)](https://github.com/SoftDevEng/aiSecurityWarden-client/actions/workflows/hermes.yml)
[![CI](https://github.com/SoftDevEng/aiSecurityWarden-client/actions/workflows/ci.yml/badge.svg)](https://github.com/SoftDevEng/aiSecurityWarden-client/actions/workflows/ci.yml)

**Keep your bots' API keys out of `.env`.** Hermes loads them at startup from an end-to-end encrypted vault. Revoke a
bot's access in one click, or require a phone approval before it gets them.

A [Hermes Agent](https://hermes-agent.nousresearch.com) secret source for [aiSecurityWarden](https://aisecuritywarden.com).
Linux, macOS and Windows. Apache-2.0.

## Why

A `.env` file is plain text. Every program running as you can read it, every backup and copy keeps it, and when a
key leaks you can't tell which bot leaked it or switch off just that one. With this plugin:

| | `.env` | aiSecurityWarden |
|---|---|---|
| Where the keys sit | Plain text on disk | Encrypted on your machines; the server stores only ciphertext, names included |
| What's on the bot's box | Every key | One machine token, scoped to that bot's project |
| A stolen copy | Works anywhere, forever | Refused from other addresses; revoked in one click; you get a push when it's tried |
| Who may start the bot with its keys | Anyone who can run it | Optionally only after your **Approve** on Telegram, ntfy, Discord or the website |
| Audit | None | Every read, approval and refusal, with its address |

## The 3 a.m. restart

The gateway crashes and systemd restarts it while you sleep. What happens:

- **Token without approval** (the usual choice for an unattended gateway, locked to the box's address): the keys load
  and the bot is back in seconds.
- **Token with phone approval**: your phone asks. If nobody answers in time, Hermes **comes up without those keys
  instead of hanging**, and the server sends a second push, *"started without secrets?"*, so the bot doesn't look
  fine until something fails later.
- **Revoked or stolen token**: refused, and you get a *"token refused"* push naming the address it came from.

## Install

1. Install the client and store a machine token (create the token and its one-time setup code on the website):

   ```bash
   pipx install aisecuritywarden
   swctl --server https://app.aisecuritywarden.com setup
   ```

2. Install the plugin from the Hermes plugin catalog. In the desktop app, search for **aisecuritywarden** under
   **Capabilities → Plugins**. Or from a terminal:

   ```bash
   hermes plugins install aisecuritywarden --enable
   ```

3. Turn the source on, then restart Hermes (the gateway and the dashboard, if they run):

   ```bash
   hermes config set secrets.aisecuritywarden.enabled true
   ```

   With one token stored on the computer that's all. With several, also name the one whose project holds Hermes's
   keys: `hermes config set secrets.aisecuritywarden.token_name <its name>`.

At start-up Hermes then prints `aiSecurityWarden: applied N secrets`, or why it couldn't and what to do. Updates: the
desktop app shows **Update to …** when a new version is listed. Keep the client current too
(`pipx upgrade aisecuritywarden`).

## What it protects, and what it doesn't

**Protects**
- **Keys at rest.** Nothing in `.env`. The vault holds ciphertext only; decryption happens on your machine with a key
  the server never sees.
- **The token.** The plugin never reads it: it runs `swctl export`, which holds the token, and gets only the secrets
  back. swctl gets its own settings and nothing else from Hermes's environment.
- **A stolen token.** Address lists, expiry, instant revoke, optional approval per start, and a push when a refused
  token is used.
- **A poisoned vault.** A secret can't redirect the plugin (Hermes never lets a source overwrite `SW_*`), and secrets
  named `LD_PRELOAD`, `PATH`, `PYTHONPATH`, `NODE_OPTIONS`, `HTTPS_PROXY`, `DYLD_*` and other names that change how
  programs load or connect are **refused** unless you list them in `allow_names`.

**Doesn't**
- **After start-up the keys are in Hermes's environment**, as with every secret source. Code running as the same user
  can read them, and the agent could repeat a value if a tool hands one back. Use a project for Hermes's own keys,
  not a connector's token. (Hermes itself keeps managed keys out of the agent's terminal and code sandboxes.)
- **A server it can't reach can't warn you.** If the network is down at start-up, only Hermes's log says so.
- **Approval guards the start, not each use.** For approval on what the agent *does* (send an email, merge a pull
  request, run a root command), see the
  [aiSecurityWarden client](https://github.com/SoftDevEng/aiSecurityWarden-client)'s connectors and `sw-mcp`.

## Tested

- **Hermes's own secret-source conformance kit**: passes against Hermes **v0.21.6, v0.23.4 and main**, on Linux and
  Windows, on every plugin change and every week
  ([Hermes compatibility](https://github.com/SoftDevEng/aiSecurityWarden-client/actions/workflows/hermes.yml)).
- **44 plugin tests**, negative cases first: malformed config never raises, a wrong-shaped or oversized answer is
  refused, only `SW_*` settings reach swctl, refused names never echo their values, each failure maps to the right
  Hermes error kind with a one-line fix.
- **`hermes plugins validate`**: passes, security scan *safe*, no core overrides. One capability only:
  `register_secret_source`.

## Settings (`secrets.aisecuritywarden` in config.yaml)

| Key | Default | |
|---|---|---|
| `enabled` | `false` | Load the secrets at start-up |
| `token_name` / `token_id` | the only stored token | Which stored token, when there are several |
| `allow_names` | `[]` | Names to load even though they change how programs load or connect |
| `timeout_seconds` | `180` | Longest wait, including a phone approval |
| `server_url` | the token's own server | Needed only for a token kept outside the client's token store |
| `swctl` | PATH, next to Hermes's Python, then `~/.local/bin` | Full path of swctl |
| `token_file` | `SW_ACCESS_TOKEN_FILE` | A token kept in a file of its own |
| `ca_bundle` | `SW_CA_BUNDLE` | Your own CA, for a private server |
| `override_existing` | `false` | Replace values already in `.env` or the shell |

More: the [Hermes guide](https://github.com/SoftDevEng/aiSecurityWarden-client/tree/main/examples/hermes) in the
client repository.

## Source

This repository is a copy of `securitywarden/hermes_plugin/` from the open-source
[aiSecurityWarden client](https://github.com/SoftDevEng/aiSecurityWarden-client), made at each client release; its
tests run there. Please open issues and pull requests on the client repository.

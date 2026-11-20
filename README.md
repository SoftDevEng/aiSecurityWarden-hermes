# aiSecurityWarden for Hermes Agent

A [Hermes Agent](https://hermes-agent.nousresearch.com) **secret source**. At start-up Hermes loads the API keys an
[aiSecurityWarden](https://aisecuritywarden.com) machine token may read: model provider keys, chat platform tokens,
keys for its MCP servers. Hermes keeps them out of the agent's terminal and code sandboxes.

- **End-to-end encrypted.** Secrets are encrypted on your machines; the server stores only ciphertext.
- **Revocable, scoped tokens.** One token per bot reads only that bot's project, can be limited to the bot's
  address, and is revoked on the website in one click.
- **Optional phone approval.** A token can require your Approve (Telegram, ntfy or the website) before each start.
- **The token never enters Hermes.** The plugin runs `swctl export` from the aiSecurityWarden client, which holds
  the token; Hermes only receives the secrets. swctl is given its own settings, nothing else from Hermes's
  environment.
- Linux, macOS and Windows.

## Install

1. Install the client and store a machine token (create the token and its one-time setup code on the website):

   ```bash
   pipx install aisecuritywarden
   swctl --server https://app.aisecuritywarden.com setup
   ```

2. Install the plugin. In the Hermes desktop app: **Capabilities → Plugins → Install from Git**, with
   `https://github.com/SoftDevEng/aiSecurityWarden-hermes`. Or from a terminal:

   ```bash
   hermes plugins install SoftDevEng/aiSecurityWarden-hermes --enable
   ```

3. Turn the source on, then restart Hermes (the gateway and the dashboard, if they run):

   ```bash
   hermes config set secrets.aisecuritywarden.enabled true
   ```

   With one token stored on the computer that's all; it's used with the server it was set up with. With several,
   also name the one whose project holds Hermes's keys:

   ```bash
   hermes config set secrets.aisecuritywarden.token_name <its name>
   ```

At start-up Hermes then says `aiSecurityWarden: applied N secrets`, or why it couldn't and what to do.

Updates: **Update all** in the desktop app, or `hermes plugins update aisecuritywarden`. Keep the client current too
(`pipx upgrade aisecuritywarden`): the plugin runs its swctl.

## Settings (`secrets.aisecuritywarden` in config.yaml)

| Key | Default | |
|---|---|---|
| `enabled` | `false` | Load the secrets at start-up |
| `token_name` / `token_id` | the only stored token | Which stored token, when there are several |
| `server_url` | the token's own server | Needed only for a token kept outside the client's token store |
| `swctl` | PATH, next to Hermes's Python, then `~/.local/bin` | Full path of swctl |
| `token_file` | `SW_ACCESS_TOKEN_FILE` | A token kept in a file of its own |
| `ca_bundle` | `SW_CA_BUNDLE` | Your own CA, for a private server |
| `timeout_seconds` | `180` | Longest wait, including a phone approval |
| `override_existing` | `false` | Replace values already in `.env` or the shell |

Every secret in the token's project goes into Hermes's environment: use a project for Hermes's own keys, not a
connector's token. A secret in the vault can't change how the plugin reaches the vault: Hermes never lets any source
overwrite `SW_SERVER_URL`, `SW_ACCESS_TOKEN`, `SW_ALLOW_HTTP` or the other `SW_*` variables swctl reads.

More: the [Hermes guide](https://github.com/SoftDevEng/aiSecurityWarden-client/tree/main/examples/hermes) in the
client repository.

## Source

This repository is a copy of `securitywarden/hermes_plugin/` from the open-source
[aiSecurityWarden client](https://github.com/SoftDevEng/aiSecurityWarden-client), made at each client release; its
tests (including Hermes's secret-source conformance kit) run there. Please open issues and pull requests on the
client repository. Apache-2.0.

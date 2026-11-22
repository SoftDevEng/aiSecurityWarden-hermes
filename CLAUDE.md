# CLAUDE.md — aiSecurityWarden-hermes

The Hermes Agent secret-source plugin for aiSecurityWarden, as Hermes installs it (`plugin.yaml` and `__init__.py`
at the top level, so `hermes plugins install SoftDevEng/aiSecurityWarden-hermes` keeps the Git history and
`hermes plugins update` / Update all work).

**Don't edit `__init__.py` or `plugin.yaml` here.** They are copies of `src/securitywarden/hermes_plugin/` in the
client repository (aiSecurityWarden-client), where the plugin is changed and tested. At each client release, from a
clone of the client at the release tag:

```bash
python -m securitywarden.client.hermes <path to this clone>
```

then commit "Plugin vX.Y.Z" here, matching the client version (`plugin.yaml` carries it). The plugin is listed in the
Hermes plugin catalog (`plugin-catalog/aisecuritywarden.yaml` in NousResearch/hermes-agent), pinned to a commit of this
repository: a changed plugin also needs a catalog PR bumping `sha` and `version` (the server repository's
`scripts/bump-hermes-catalog.ps1`). Never force-push or rewrite `main`: a pinned commit that disappears gets the
listing removed. The full order is in the server repository's `docs/RELEASE.md`. Sessions open PRs only; Brian
merges.

Keep nothing else at the top level that Hermes would load: no other Python files, no tests (they live in the client).
